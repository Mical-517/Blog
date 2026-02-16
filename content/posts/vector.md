---
title: Vector
published: 2026-02-16T09:58:11+08:00
summary: "关于vector使用"
cover:
  image: https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202602011340604.png
tags: [vector]
categories: '现代c++'
draft: false
lang: ''
---

#### . 基础准备（头文件 + 命名空间）

使用 vector 前必须包含头文件，处理命名空间时遵循 "先简单后精确" 的原则：





```c++
// 必备头文件
#include <vector>
// 基础IO，方便后续打印测试
#include <iostream>

// 新手推荐：简化代码，避免频繁写std::
using namespace std;

// 进阶做法（冲突时使用）：显式限定
// std::vector<int> v;
```

#### 2. 核心定义语法（对比传统数组）

vector 是模板类，定义时必须指定元素类型，核心语法和数组的区别如下：







```c++
int main() {
    // 1. 空vector（最常用）
    vector<int> v1;

    // 2. 初始化指定容量+默认值（圆括号是构造函数）
    // 创建10个int元素，每个初始值为0
    vector<int> v2(10);
    // 创建5个double元素，每个初始值为3.14
    vector<double> v3(5, 3.14);

    // 3. 直接初始化元素（列表初始化，C++11及以上）
    vector<string> v4{"apple", "banana", "orange"};

    // ❌ 错误示范：用[]设定容量（会被解析为数组的数组）
    // vector<int> v5[10]; // 这是"包含10个vector<int>的数组"，不是"容量为10的vector"

    // ✅ 传统数组对比：int arr[10]; // 固定大小，无法动态扩容
    return 0;
}
```

#### 3. 模板机制（理解 "类型定制"）

vector 的`<元素类型>`是模板参数，编译器会为不同类型生成独立代码，支持嵌套实现多维结构：







```c++
// 不同元素类型 = 不同的vector类型
vector<int> v_int;    // 存储int的vector
vector<float> v_float;// 存储float的vector
vector<string> v_str; // 存储字符串的vector

// 嵌套：二维vector（相当于"变长的二维数组"）
// 外层vector的每个元素都是一个vector<int>
vector<vector<int>> v_2d = {{1,2}, {3,4,5}, {6}};
```

### 二、vector 核心操作（CRUD + 访问）

#### 1. 数据访问（3 种方式）





```c++
#include <vector>
#include <iostream>
using namespace std;

int main() {
    vector<int> v = {10, 20, 30, 40, 50};

    // 方式1：下标访问（和数组一致，⚠️ 不做越界检查）
    cout << v[2] << endl; // 输出30
    // v[10] = 100; // ❌ 越界，编译不报错，运行可能崩溃

    // 方式2：at()访问（✅ 有越界检查，更安全）
    cout << v.at(3) << endl; // 输出40
    // v.at(10) = 100; // ❌ 越界，直接抛异常，便于调试

    // 方式3：范围for循环（C++11+，简洁安全）
    cout << "范围for遍历：";
    for (int num : v) {
        cout << num << " ";
    }
    cout << endl;

    // 方式4：迭代器（通用遍历方式，适配所有STL容器）
    cout << "迭代器遍历：";
    for (vector<int>::iterator it = v.begin(); it != v.end(); ++it) {
        cout << *it << " "; // *it 解引用获取元素值
    }
    cout << endl;

    return 0;
}
```

输出结果：





```c++
30
40
范围for遍历：10 20 30 40 50 
迭代器遍历：10 20 30 40 50 
```

#### 2. 增加元素（push_back/insert）



```c++
#include <vector>
#include <iostream>
using namespace std;

int main() {
    vector<int> v = {1, 2, 3};

    // 1. 末尾添加（push_back，效率高）
    v.push_back(4); // v: {1,2,3,4}
    v.push_back(5); // v: {1,2,3,4,5}

    // 2. 中间插入（insert，需指定迭代器位置）
    // 在第2个元素（索引1）位置插入10
    // v.begin()是起始迭代器，+1移动到第2个元素位置
    v.insert(v.begin() + 1, 10); // v: {1,10,2,3,4,5}

    // 批量插入：在末尾插入3个9
    v.insert(v.end(), 3, 9); // v: {1,10,2,3,4,5,9,9,9}

    // 遍历验证
    for (int num : v) {
        cout << num << " ";
    } // 输出：1 10 2 3 4 5 9 9 9

    return 0;
}
```

#### 3. 删除元素（erase/pop_back）



```c++
#include <vector>
#include <iostream>
using namespace std;

int main() {
    vector<int> v = {1,10,2,3,4,5,9,9,9};

    // 1. 删除末尾元素（pop_back，效率高）
    v.pop_back(); // v: {1,10,2,3,4,5,9,9}

    // 2. 删除指定位置元素（erase，迭代器参数）
    // 删除第2个元素（索引1）
    v.erase(v.begin() + 1); // v: {1,2,3,4,5,9,9}

    // 3. 删除区间元素（左闭右开）：删除索引2到4的元素（3,4,5）
    v.erase(v.begin() + 2, v.begin() + 5); // v: {1,2,9,9}

    // 遍历验证
    for (int num : v) {
        cout << num << " ";
    } // 输出：1 2 9 9

    return 0;
}
```

#### 4. 查找元素（find + 迭代器）

需包含`<algorithm>`头文件，find 是 STL 算法，遵循 "函数式编程" 理念：



```c++
#include <vector>
#include <iostream>
#include <algorithm> // find函数的头文件
using namespace std;

int main() {
    vector<int> v = {1, 2, 3, 2, 4, 2, 5};
    int target = 2;

    // 第一次查找：从begin()到end()找2
    vector<int>::iterator it = find(v.begin(), v.end(), target);
    if (it != v.end()) {
        cout << "找到第一个" << target << "，位置：" << it - v.begin() << endl; // 输出：位置1
    }

    // 多次查找：从上次找到的位置+1继续找
    while (true) {
        it = find(it + 1, v.end(), target);
        if (it == v.end()) break;
        cout << "找到下一个" << target << "，位置：" << it - v.begin() << endl;
    }
    // 输出：位置3 → 位置5

    return 0;
}
```

### 三、vector 内存管理（核心注意事项）

#### 1. 动态扩容规则

- 扩容触发条件：只有调用`push_back()`/`insert()`等插入函数，且当前容量不足时才会扩容；

- 扩容原理：vector 会申请一块更大的内存（通常是原容量的 1.5~2 倍），把原数据拷贝过去，释放旧内存；

- 下标直接赋值不扩容：`vector<int> v; v[0] = 10;` ❌ 越界，因为 v 是空的，没有扩容；

- 手动控制容量：

  

  

  

  

  ```c++
  vector<int> v;
  v.reserve(100); // 预分配100个元素的空间（仅预留，不初始化）
  v.resize(50);   // 调整大小为50，新增元素默认初始化（int为0）
  cout << v.capacity() << endl; // 输出100（容量）
  cout << v.size() << endl;     // 输出50（实际元素数）
  ```

  

#### 2. 效率注意事项

- `push_back()` 末尾操作效率高（扩容除外），`insert()`/`erase()` 中间操作效率低（需移动元素）；
- 若已知元素数量，提前用`reserve()`预分配容量，避免频繁扩容拷贝；
- 迭代器失效：扩容 / 删除元素后，原有迭代器可能失效（指向旧内存），需重新获取。

### 四、vector 常用成员函数（速查）



|     函数     |                   作用                   |
| :----------: | :--------------------------------------: |
|   `size()`   |             返回当前元素个数             |
| `capacity()` | 返回当前分配的容量（可存储的最大元素数） |
|  `empty()`   |        判断是否为空（size ()==0）        |
|  `clear()`   | 清空所有元素（size ()=0，capacity 不变） |
|  `begin()`   |        返回指向第一个元素的迭代器        |
|   `end()`    |  返回指向最后一个元素**下一位**的迭代器  |
|  `front()`   |   返回第一个元素（等价于 * begin ()）    |
|   `back()`   |             返回最后一个元素             |

### 总结

1. **核心特性**：vector 是 STL 动态数组，支持自动扩容，模板化支持任意元素类型，嵌套可实现多维结构；
2. **关键操作**：访问用下标 /at/ 迭代器（at 更安全），增删用 push_back/insert/erase（末尾操作效率更高），查找用 find + 迭代器（函数式编程更优）；
3. **注意事项**：下标直接赋值不扩容（易越界），中间插入 / 删除效率低，扩容会导致迭代器失效，提前 reserve 可优化性能。