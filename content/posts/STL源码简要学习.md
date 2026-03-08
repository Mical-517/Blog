---
title: STL源码简要学习
published: 2026-03-08T15:20:41+08:00
summary: "介绍STL源码设计的各种知识"
cover:
  image: https://img-blog.csdnimg.cn/20190921113703683.png?x-oss-process=image/resize,m_fixed,h_224,w_224
tags: [STL]
categories: '数据结构'
draft: false
lang: ''
---

# 整体认知

![](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202603071804628.png)







# 前置知识

## 1.模版的泛化，特化，偏特化

### 1. 模板泛化（通用模板）

**泛化**就是模板的**通用版本**，不针对任何特定类型，是模板最基础的形态。你定义一个模板时，默认写的就是泛化版本，它能适配所有符合语法的类型。

#### 代码示例（函数模板泛化）

```c++
#include <iostream>
using namespace std;

// 模板泛化：通用版本，适用于所有类型
template <typename T>
T add(T a, T b) {
    cout << "通用版本：";
    return a + b;
}

int main() {
    // 使用泛化版本：int 类型
    cout << add(1, 2) << endl;  // 输出：通用版本：3
    // 使用泛化版本：double 类型
    cout << add(1.5, 2.5) << endl;  // 输出：通用版本：4
    return 0;
}
```

#### 代码示例（类模板泛化）



```c++
template <typename T>
class MyContainer {
public:
    MyContainer(T val) : value(val) {}
    void print() {
        cout << "通用容器：" << value << endl;
    }
private:
    T value;
};

// 使用泛化版本
MyContainer<int> c1(10);    // 通用容器：10
MyContainer<string> c2("hello");  // 通用容器：hello
```

**核心特点**：泛化版本是 “兜底” 的，只要没有更匹配的特化 / 偏特化版本，就会调用它。

### 2. 模板特化（全特化）

**特化**（也叫全特化）是为**某个具体的类型**完全定制模板的实现，相当于给特定类型 “开小灶”。特化时，模板参数被**完全固定**，不再是泛型。

#### 适用场景

当通用版本对某个类型不适用（比如逻辑错误、效率低），需要单独实现时，就用特化。

#### 代码示例（函数模板特化）





```c++
// 泛化版本（同上）
template <typename T>
T add(T a, T b) {
    cout << "通用版本：";
    return a + b;
}

// 对 const char* 类型的全特化（处理字符串拼接）
template <>
const char* add<const char*>(const char* a, const char* b) {
    cout << "特化版本（字符串）：";
    // 简单拼接（实际开发需注意内存管理，这里仅示例）
    static char res[100];
    strcpy(res, a);
    strcat(res, b);
    return res;
}

int main() {
    cout << add(1, 2) << endl;  // 通用版本：3
    cout << add("hello", "world") << endl;  // 特化版本（字符串）：helloworld
    return 0;
}
```

#### 代码示例（类模板全特化）





```c++
// 泛化版本（同上）
template <typename T>
class MyContainer {
public:
    MyContainer(T val) : value(val) {}
    void print() {
        cout << "通用容器：" << value << endl;
    }
private:
    T value;
};

// 对 bool 类型的全特化
template <>
class MyContainer<bool> {
public:
    MyContainer(bool val) : value(val) {}
    void print() {
        // 定制输出逻辑：把 bool 转成 "真/假"
        cout << "布尔容器：" << (value ? "真" : "假") << endl;
    }
private:
    bool value;
};

int main() {
    MyContainer<int> c1(10);    // 通用容器：10
    MyContainer<bool> c2(true); // 布尔容器：真
    return 0;
}
```

**核心特点**：特化版本的模板参数被**完全确定**（比如 `const char*`、`bool`），是 “精准匹配”。

### 3. 模板偏特化（部分特化）

**偏特化**只针对**模板参数的一部分**进行定制，剩下的参数仍保持泛型。它只适用于**类模板**（函数模板不支持偏特化）。

#### 适用场景

当需要为 “某一类类型”（比如指针类型、引用类型、特定数量的模板参数）定制模板，而非单个具体类型时，用偏特化。

#### 代码示例 1：针对指针类型的偏特化



```c++
// 泛化版本
template <typename T>
class MyContainer {
public:
    MyContainer(T val) : value(val) {}
    void print() {
        cout << "通用容器：" << value << endl;
    }
private:
    T value;
};

// 偏特化：针对 T*（指针类型）
template <typename T>
class MyContainer<T*> {
public:
    MyContainer(T* val) : value(val) {}
    void print() {
        // 定制逻辑：输出指针地址 + 指向的值
        cout << "指针容器：地址=" << value << "，值=" << *value << endl;
    }
private:
    T* value;
};

int main() {
    int num = 100;
    MyContainer<int> c1(10);    // 通用容器：10
    MyContainer<int*> c2(&num); // 指针容器：地址=0x7ffeefbff5ac，值=100
    return 0;
}
```

#### 代码示例 2：针对多个模板参数的偏特化



```c++
// 泛化版本：两个模板参数
template <typename T, typename U>
class Pair {
public:
    Pair(T a, U b) : first(a), second(b) {}
    void print() {
        cout << "通用 Pair：" << first << ", " << second << endl;
    }
private:
    T first;
    U second;
};

// 偏特化：第二个参数固定为 int
template <typename T>
class Pair<T, int> {
public:
    Pair(T a, int b) : first(a), second(b) {}
    void print() {
        cout << "偏特化 Pair（第二个参数是int）：" << first << ", " << second << endl;
    }
private:
    T first;
    int second;
};

int main() {
    Pair<string, double> p1("a", 3.14);  // 通用 Pair：a, 3.14
    Pair<string, int> p2("b", 100);      // 偏特化 Pair（第二个参数是int）：b, 100
    return 0;
}
```

**核心特点**：偏特化是 “部分匹配”，模板参数仍有泛型部分，适配某一类类型而非单个类型。

------

### 总结

1. **泛化**：模板的通用版本，适配所有符合语法的类型，是模板的基础形态。
2. **特化（全特化）**：为**单个具体类型**定制模板实现，模板参数完全固定，仅适用于该类型。
3. **偏特化**：仅针对模板参数的**一部分**定制（如指针类型、固定某一个参数），仅支持类模板，适配 “某一类类型”



## 2.关于traits（萃取作用）

从整体认知，我们知道，关于容器与泛型算法是如何交互的，是通过迭代器来进行交互，这一过程中有问题：

1. 泛型算法接受任意类型，他怎么区分接受的是一个指针还是迭代器

2. 泛型算法接受迭代器之后，我们知道迭代器本身具有5类型：（主要是下面三个）

   1. 迭代器指向的类型
   2. 迭代器的分类类型：是双向还是单向
   3. 两个迭代器之间距离的类型

   算法至少要知道这三种类型才能够清楚改使用什么策略来实现算法，**他是怎么知道的呢？**

结论：迭代器与算法之间依靠一个中间类（**就是traits**，它的作用就是提取迭代器或者指针的类型，然后告诉算法）

## 前置知识代码示例

```c++
#pragma once
#include <memory>
#include <iostream> // For cout in print()
#include <iterator> // For iterator traits
using namespace std;

template <class DataType>
class DLinkList;

template <class T>
class MyIterator;

template <class DataType>
class Node
{
public:
    Node() = default;
    Node(const DataType &infor, Node<DataType> *pre, std::unique_ptr<Node<DataType>> next)
    {
        this->infor = infor;
        this->pre = pre;
        this->next = move(next);
    }

private:
    // 受到智能指针特性的限制，一块堆内存智能有一个智能指针管理，所以node成员有一个智能指针用于管理，一个裸指针用于观测
    DataType infor;
    std::unique_ptr<Node<DataType>> next;
    Node<DataType> *pre{nullptr};

    friend class DLinkList<DataType>;
    friend class MyIterator<DataType>;
};

// 自定义list的迭代器类型
template <class T>
class MyIterator : public std::iterator<std::bidirectional_iterator_tag, T>
{
public:
    // 添加构造函数
    MyIterator(Node<T> *p = nullptr) : ptr(p) {}

    // 重载*，返回指向对象的引用
    T &operator*() const
    {
        return ptr->infor; // 修正：返回节点的数据
    }

    // 建议增加 -> 运算符
    T *operator->() const
    {
        return &(ptr->infor);
    }

    // 前置++，返回迭代器本身的引用
    MyIterator<T> &operator++()
    {
        ptr = ptr->next.get();
        return *this;
    }

    // 后置++,返回迭代器自身拷贝
    MyIterator<T> operator++(int)
    {
        MyIterator<T> temp = *this; // 修正：创建当前迭代器的副本
        ptr = ptr->next.get();
        return temp; // 修正：返回副本
    }

    // 添加 operator-- 以支持双向迭代
    MyIterator<T> &operator--()
    {
        ptr = ptr->pre;
        return *this;
    }

    MyIterator<T> operator--(int)
    {
        MyIterator<T> temp = *this;
        ptr = ptr->pre;
        return temp;
    }

    // 添加比较运算符
    bool operator==(const MyIterator<T> &other) const
    {
        return ptr == other.ptr;
    }

    bool operator!=(const MyIterator<T> &other) const
    {
        return ptr != other.ptr;
    }

private:
    Node<T> *ptr{nullptr};
};

template <class DataType>
class DLinkList
{
public:
    // 定义迭代器类型别名
    using iterator = MyIterator<DataType>;

    // 默认构造函数
    DLinkList()
    {
        this->head = make_unique<Node<DataType>>();
        this->head->next = make_unique<Node<DataType>>();
        this->tail = this->head->next.get();
        tail->pre = this->head.get();
    }

    // 删除成员变量 iterator，替换为 begin() 和 end() 方法
    iterator begin()
    {
        return iterator(head->next.get());
    }

    iterator end()
    {
        return iterator(tail);
    }

    // 拷贝构造函数
    DLinkList(const DLinkList &other);
    // 析构函数
    ~DLinkList() = default;
    // 前插入
    void push_front(const DataType &data);
    // 后插入
    void push_back(const DataType &data);
    // 打印所有数据
    void print();

private:
    std::unique_ptr<Node<DataType>> head;
    Node<DataType> *tail{nullptr}; // 仅用来观测
};

// 拷贝构造函数
template <class DataType>
DLinkList<DataType>::DLinkList(const DLinkList &other) : DLinkList()
{
    // 深拷贝
    // 行动指针遍历other,应该用普通指针遍历，因为智能指针遍历是会释放空间
    Node<DataType> *otherPtr = other.head->next.get();
    Node<DataType> *currPtr = this->head.get();
    while (otherPtr != other.tail)
    {
        // 创建新节点，将当前节点的下个节点的所有权交给当前节点的next
        auto newNode = make_unique<Node<DataType>>(otherPtr->infor, currPtr, move(currPtr->next));
        tail->pre = newNode.get();
        // 转移当前节点所有权
        currPtr->next = move(newNode);
        // 更新行动指针
        otherPtr = otherPtr->next.get();
        currPtr = currPtr->next.get();
    }
}

// 前插入
template <class DataType>
void DLinkList<DataType>::push_front(const DataType &data)
{
    auto newNode = make_unique<Node<DataType>>(data, this->head.get(), move(head->next));
    newNode->next->pre = newNode.get();
    head->next = move(newNode);
}

// 后插入
template <class DataType>
void DLinkList<DataType>::push_back(const DataType &data)
{
    auto ptr = make_unique<Node<DataType>>(data, this->tail->pre, move(this->tail->pre->next));
    this->tail->pre = ptr.get();
    ptr->pre->next = move(ptr);
}

// 打印
template <class DataType>
void DLinkList<DataType>::print()
{
    // 使用迭代器进行打印
    for (iterator it = begin(); it != end(); ++it)
    {
        cout << *it << " ";
    }
    // 修正：加上换行
    cout << endl;
}

```

```c++
#include <iostream>
#include "DLinList-unique_ptr.h"
#include <vector>
using namespace std;

// 这是一个泛型函数，可以计算序列中元素的总和
// 它需要知道元素的类型（value_type）来初始化 sum 变量
template <class InputIterator>
typename std::iterator_traits<InputIterator>::value_type
sum_sequence(InputIterator first, InputIterator last)
{
    // 使用 traits 获取 value_type 来声明 sum 变量
    typename std::iterator_traits<InputIterator>::value_type sum = 0;
    for (; first != last; ++first)
    {
        sum = sum + *first;
    }
    return sum;
}

int main()
{
    // 1. 测试你的自定义迭代器
    DLinkList<int> myList;
    myList.push_back(10);
    myList.push_back(20);
    myList.push_back(30);

    cout << "Sum of myList: " << sum_sequence(myList.begin(), myList.end()) << endl; // 输出 60

    // 2. 测试原生指针
    int arr[] = {1, 2, 3, 4, 5};
    cout << "Sum of array: " << sum_sequence(arr, arr + 5) << endl; // 输出 15

    // 3. 测试标准库迭代器 (例如 vector)
    std::vector<int> myVec = {100, 200};
    cout << "Sum of vector: " << sum_sequence(myVec.begin(), myVec.end()) << endl; // 输出 300

    return 0;
}

```

```c++
2. iterator_traits 如何“萃取”你的 MyIterator
现在，假设有一个标准算法（比如 std::sort 或你自己写的泛型函数）拿到了你的 MyIterator，它想知道这个迭代器的特性。它不会直接去访问 MyIterator::value_type，而是会通过 iterator_traits 这个中介。

它会这样做：
// 泛型算法内部（示意代码）
template <class SomeIterator>
void some_generic_algorithm(SomeIterator begin, SomeIterator end) {
    // 算法需要知道迭代器指向的类型，以便创建一个临时变量
    typename std::iterator_traits<SomeIterator>::value_type temp_value = *begin;
    
    // 算法需要知道迭代器的类别，以选择最高效的移动策略
    // ... 比如，如果是随机访问迭代器，就可以直接 begin + 5
    // ... 如果是双向迭代器，就只能 ++ ++ ++ ++ ++
}
当 SomeIterator 是你的 MyIterator<int> 时，std::iterator_traits<MyIterator<int>> 会被实例化。它会匹配到 iterator_traits 的泛化版本（因为 MyIterator 是一个 class，不是指针）：
// iterator_traits 的泛化版本
template <class I>
struct iterator_traits {
    typedef typename I::iterator_category iterator_category;
    typedef typename I::value_type        value_type;
    typedef typename I::difference_type   difference_type;
    typedef typename I::pointer           pointer;
    typedef typename I::reference         reference;
};
这个泛化版本做的事情正如你的笔记所说：“你问我，我问他”。

std::iterator_traits 被询问 value_type。
它去询问 MyIterator 的 value_type。
因为你继承了 std::iterator，MyIterator 内部有 typedef T value_type;。
最终，算法成功得到了类型 T（比如 int）。
这就是 traits 机制的完美协作：你的迭代器提供了信息，traits 作为中介负责提取，泛型算法通过中介获取信息并正确工作。

3. 如果迭代器是原生指针会怎样？
现在对比一下，如果一个算法被传入一个 int* 指针。
int arr[] = {1, 2, 3};
int* begin = arr;
int* end = arr + 3;
// some_generic_algorithm(begin, end);
此时，SomeIterator 是 int*。std::iterator_traits<int*> 会被实例化。这时，它不会匹配泛化版本（因为 int* 没有 ::value_type 这样的成员），而是会匹配到你笔记中提到的指针的偏特化版本：
// iterator_traits 对指针的偏特化版本
template <class T>
struct iterator_traits<T*> {
    typedef std::random_access_iterator_tag iterator_category;
    typedef T                               value_type;
    typedef ptrdiff_t                       difference_type;
    typedef T*                              pointer;
    typedef T&                              reference;
};
这个特化版本直接“回答”了所有问题：

value_type 就是 T (这里是 int)。
iterator_category 是 random_access_iterator_tag（因为指针支持 p+n 这样的随机访问）。
结论： 通过泛化和特化，iterator_traits 用一套统一的接口 std::iterator_traits<It>::value_type，同时解决了自定义迭代器和原生指针这两种完全不同的类型。
```



