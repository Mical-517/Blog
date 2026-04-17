---
title: STL基础使用
published: 2026-03-07T17:58:16+08:00
summary: "简要介绍标准库中的容器分类以及使用"
cover:
  image: https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202603071802289.png
tags: [STL]
categories: 'STL'
draft: false
lang: ''
---

# 一.STL的六大部件

![image-20260307180449500](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202603071804628.png)

# 二.STL基本认知与使用

## 一、序列容器（Sequence Containers）

元素按线性顺序存储，位置由插入顺序决定。

### 1. `std::vector`

#### 实现基础

基于**动态数组**，元素存储在连续内存块中，支持动态扩容。

#### 特点与规则

- **随机访问**：通过索引（`[]`/`at()`）O (1) 时间访问元素。
- **动态扩容**：容量不足时自动重新分配内存（通常扩容 1.5~2 倍），可能导致迭代器 / 指针失效。
- **尾部操作高效**：`push_back`/`pop_back`为摊销 O (1)；中间 / 头部插入 / 删除需移动元素，O (n)。
- **缓存友好**：内存连续，遍历效率高。

#### 常用函数及参数

- **构造**：`vector<T> v(n, val)`（初始化 n 个 val）、`vector<T> v(v2)`（拷贝构造）。
- **访问**：`operator[]`（无边界检查）、`at()`（越界抛异常）、`front()`/`back()`（首尾元素）。
- **容量**：`size()`（元素数）、`capacity()`（已分配容量）、`reserve(n)`（预留容量）、`resize(n, val)`（调整大小）。
- **修改**：`push_back(val)`（尾部插入）、`pop_back()`（尾部删除）、`insert(pos, val)`（pos 前插入）、`erase(pos)`（删除 pos 处元素）、`clear()`（清空元素）。

#### 使用场景

需要快速随机访问、主要在尾部增删元素的场景（如动态数组、数据缓存）。

### 2. `std::deque`（双端队列）

#### 实现基础

基于**分段连续内存**（中央控制块 + 多个固定大小缓冲区），支持头尾高效操作。

#### 特点与规则

- **随机访问**：O (1) 时间，但比`vector`稍慢（需两次指针解引用）。
- **头尾操作高效**：`push_front`/`push_back`/`pop_front`/`pop_back`均为摊销 O (1)。
- **中间插入 / 删除**：O (n)，效率低于`list`但可能优于`vector`。
- **迭代器失效**：头尾插入可能使迭代器失效，中间插入会使所有迭代器失效。

#### 常用函数及参数

- **访问**：`operator[]`、`at()`、`front()`/`back()`。
- **修改**：`push_front(val)`（头部插入）、`pop_front()`（头部删除）、`insert()`/`erase()`。

#### 使用场景

需要在头尾频繁增删元素、同时需要随机访问的场景（如双端任务队列）。

### 3. `std::list`（双向链表）

#### 实现基础

基于**双向链表**，每个节点包含数据、前驱指针、后继指针。

#### 特点与规则

- **不支持随机访问**：访问元素需遍历，O (n)。
- **任意位置插入 / 删除高效**：O (1) 时间，仅需修改指针。
- **迭代器稳定**：插入 / 删除不影响其他元素的迭代器（仅被删元素迭代器失效）。
- **内存开销大**：每个元素需额外存储两个指针。

#### 常用函数及参数

- **修改**：`push_front()`/`push_back()`、`pop_front()`/`pop_back()`、`insert(pos, val)`、`erase(pos)`。
- **链表操作**：`splice(pos, other)`（合并链表）、`merge(other)`（合并有序链表）、`sort()`（链表排序）、`reverse()`（反转）。

#### 使用场景

需要频繁在任意位置增删元素、对迭代器稳定性要求高的场景（如频繁插入删除的数据集）。

### 4. `std::forward_list`（单向链表，C++11）

#### 实现基础

基于**单向链表**，每个节点仅含数据和后继指针，比`list`节省内存。

#### 特点与规则

- **仅支持单向遍历**：迭代器为前向迭代器（无`--`操作）。
- **插入 / 删除高效**：在指定位置后插入 / 删除为 O (1)（提供`insert_after`/`erase_after`）。
- **无`size()`函数**：获取元素数量需遍历，O (n)。

#### 常用函数及参数

- **修改**：`push_front(val)`、`pop_front()`、`insert_after(pos, val)`、`erase_after(pos)`。

#### 使用场景

对内存敏感、仅需单向遍历的场景（如轻量级链表）。

### 5. `std::array`（固定大小数组，C++11）

#### 实现基础

封装**原生固定大小数组**，大小在编译时确定（模板参数`N`）。

#### 特点与规则

- **大小固定**：不能动态扩容，编译期已知大小。
- **随机访问**：O (1) 时间，内存连续，缓存友好。
- **无动态内存管理**：存储在栈或静态存储区。

#### 常用函数及参数

- **构造**：`array<T, N> arr = {val1, val2,...}`（列表初始化）。
- **访问**：`operator[]`、`at()`、`data()`（返回底层数组指针）。
- **修改**：`fill(val)`（所有元素设为 val）、`swap(other)`（交换元素）。

#### 使用场景

需要固定大小数组、追求性能的场景（如替代原生数组）。

## 二、有序关联容器（基于红黑树，自动排序）

元素按键值自动排序，底层为红黑树，平均和最坏时间复杂度均为 O (log n)。

### 1. `std::set`

#### 实现基础

红黑树，存储**唯一键**，元素自动按升序（默认`std::less<T>`）排列。

#### 特点与规则

- **元素唯一**：插入重复元素会失败。
- **自动排序**：遍历结果有序。
- **元素不可修改**：元素为`const`，修改需先删除再插入。

#### 常用函数及参数

- **插入**：`insert(val)`（返回`pair<iterator, bool>`，bool 表示是否成功）。
- **删除**：`erase(val)`（删除所有等于 val 的元素，返回数量）、`erase(pos)`。
- **查找**：`find(val)`（返回迭代器，未找到返回`end()`）、`count(val)`（返回 0 或 1）。
- **范围**：`lower_bound(val)`（第一个不小于 val 的迭代器）、`upper_bound(val)`（第一个大于 val 的迭代器）。

#### 使用场景

需要存储唯一元素、自动排序、高效查找的场景（如去重排序、集合运算）。

### 2. `std::multiset`

#### 实现基础

红黑树，与`set`类似，但**允许键重复**。

#### 特点与规则

- **元素可重复**：可插入多个相同值的元素。
- **自动排序**：同`set`。

#### 常用函数及参数

- **插入**：`insert(val)`（返回迭代器，无需返回 bool）。
- **删除**：`erase(val)`（删除所有等于 val 的元素，返回数量）。
- **查找**：`count(val)`（可返回大于 1 的值）。

#### 使用场景

需要存储可重复元素、自动排序的场景（如统计频率）。

### 3. `std::map`

#### 实现基础

红黑树，存储**键值对**（`std::pair<const Key, T>`），按键自动排序，键唯一。

#### 特点与规则

- **键值对存储**：`first`为键（`const`），`second`为值。
- **键唯一**：插入重复键会失败。
- **键不可修改，值可修改**：通过迭代器可修改值（`second`）。

#### 常用函数及参数

- **插入**：`insert({key, val})`、`emplace(key, val)`（原地构造，更高效）。
- **访问**：`operator[](key)`（若 key 不存在则插入默认值）、`at(key)`（越界抛异常）。
- **删除**：`erase(key)`（删除键为 key 的元素，返回 0 或 1）。
- **查找**：`find(key)`、`count(key)`。

#### 使用场景

需要键值对映射、按键排序、高效查找的场景（如字典、索引）。

### 4. `std::multimap`

#### 实现基础

红黑树，与`map`类似，但**允许键重复**。

#### 特点与规则

- **键可重复**：可插入多个相同键的键值对。
- **无`operator[]`和`at()`**：因键不唯一，无法通过单个键访问值。

#### 常用函数及参数

- **插入**：`insert({key, val})`（返回迭代器）。
- **删除**：`erase(key)`（删除所有键为 key 的元素，返回数量）。
- **查找**：`equal_range(key)`（返回键为 key 的元素范围）。

#### 使用场景

需要一键多值映射、按键排序的场景（如一对多关系）。

## 三、无序关联容器（基于哈希表，C++11）

元素由哈希值组织，无序，平均时间复杂度 O (1)，最坏 O (n)（哈希冲突严重时）。

### 1. `std::unordered_set`

#### 实现基础

哈希表，存储**唯一键**，无序。

#### 特点与规则

- **无序**：元素顺序由哈希值和桶分布决定。
- **极高效操作**：查找 / 插入 / 删除平均 O (1)。
- **需哈希函数**：默认使用`std::hash<Key>`（自定义类型需特化）。

#### 常用函数及参数

- **哈希表操作**：`bucket_count()`（桶数量）、`load_factor()`（加载因子：元素数 / 桶数）、`rehash(n)`（重新设置桶数量）。

#### 使用场景

对查找 / 插入 / 删除速度要求极高、不关心顺序的场景（如快速去重）。

### 2. `std::unordered_multiset`

#### 实现基础

哈希表，与`unordered_set`类似，但**允许键重复**。

#### 使用场景

需要快速存储可重复元素、不关心顺序的场景（如快速统计频率）。

### 3. `std::unordered_map`

#### 实现基础

哈希表，存储**键值对**，键唯一，无序。

#### 特点与规则

- **极高效键值查找**：通过键操作平均 O (1)。
- **键不可修改，值可修改**：同`map`。

#### 常用函数及参数

- **访问**：`operator[](key)`、`at(key)`。
- **哈希表操作**：同`unordered_set`。

#### 使用场景

需要快速键值对查找、不关心键顺序的场景（如哈希表、缓存）。

### 4. `std::unordered_multimap`

#### 实现基础

哈希表，与`unordered_map`类似，但**允许键重复**。

#### 使用场景

需要快速一键多值查找、不关心键顺序的场景（如快速一对多关系）。

## 四、容器适配器（封装底层容器，提供受限接口）

适配器不直接存储元素，而是封装底层容器（如`deque`/`vector`/`list`），提供特定接口。

### 1. `std::stack`（栈）

#### 实现基础

默认基于`std::deque`（也可使用`vector`/`list`，需支持`push_back`/`pop_back`/`back()`）。

#### 特点与规则

- **后进先出（LIFO）**：仅允许在栈顶（`top`）插入、删除、访问。
- **无迭代器**：不支持遍历。

#### 常用函数及参数

- **操作**：`push(val)`（栈顶插入）、`pop()`（栈顶删除）、`top()`（栈顶元素）。
- **容量**：`empty()`、`size()`。

#### 使用场景

需要 LIFO 行为的场景（如函数调用栈、表达式求值、括号匹配、DFS）。

### 2. `std::queue`（队列）

#### 实现基础

默认基于`std::deque`（也可使用`list`，需支持`push_back`/`pop_front`/`front()`/`back()`；不能用`vector`，因无高效`pop_front`）。

#### 特点与规则

- **先进先出（FIFO）**：仅允许在队尾（`back`）插入，队头（`front`）删除、访问。
- **无迭代器**：不支持遍历。

#### 常用函数及参数

- **操作**：`push(val)`（队尾插入）、`pop()`（队头删除）、`front()`/`back()`（队头 / 队尾元素）。

#### 使用场景

需要 FIFO 行为的场景（如任务队列、消息队列、BFS）。

### 3. `std::priority_queue`（优先级队列）

#### 实现基础

默认基于`std::vector`，底层用**堆算法**实现（默认大顶堆）。

#### 特点与规则

- **按优先级排序**：队头（`top`）为优先级最高的元素（默认最大元素）。
- **无迭代器**：不支持遍历。
- **插入 / 删除**：插入 O (log n)，删除队头 O (log n)，访问队头 O (1)。

#### 常用函数及参数

- **构造**：`priority_queue<T, vector<T>, greater<T>> pq`（小顶堆，需指定`greater<T>`）。
- **操作**：`push(val)`、`pop()`、`top()`。

#### 使用场景

需要按优先级处理元素的场景（如任务调度、Dijkstra 算法、哈夫曼编码）。

## 容器选择指南



|        需求场景         |          推荐容器          |
| :---------------------: | :------------------------: |
| 快速随机访问 + 尾部增删 |       `std::vector`        |
| 头尾频繁增删 + 随机访问 |        `std::deque`        |
|    任意位置频繁增删     |        `std::list`         |
|   唯一元素 + 自动排序   |         `std::set`         |
|    键值对 + 自动排序    |         `std::map`         |
|  极快查找 + 不关心顺序  | `std::unordered_set`/`map` |
|    后进先出（LIFO）     |        `std::stack`        |
|    先进先出（FIFO）     |        `std::queue`        |
|      按优先级处理       |   `std::priority_queue`    |

# STL使用基本代码

## 一、序列容器

### 1. `std::vector`（动态数组）



```c++
#include <vector>
#include <iostream>

int main() {
    // 1. 构造
    std::vector<int> v1;                 // 空vector
    std::vector<int> v2(5, 10);         // 5个10
    std::vector<int> v3 = {1, 2, 3, 4}; // 列表初始化

    // 2. 插入/删除
    v1.push_back(10); // 尾部插入
    v1.insert(v1.begin(), 20); // 头部插入
    v1.pop_back(); // 尾部删除

    // 3. 访问
    std::cout << v2[0] << " " << v2.at(1) << "\n"; // 索引访问
    std::cout << "首元素: " << v3.front() << " 尾元素: " << v3.back() << "\n";

    // 4. 遍历
    for (int num : v3) std::cout << num << " ";
    return 0;
}
```

### 2. `std::deque`（双端队列）



```c++
#include <deque>
#include <iostream>

int main() {
    std::deque<int> dq = {1, 2, 3};

    // 1. 头尾操作（核心特点）
    dq.push_front(0); // 头部插入
    dq.push_back(4);  // 尾部插入
    dq.pop_front();   // 头部删除
    dq.pop_back();    // 尾部删除

    // 2. 随机访问
    std::cout << dq[0] << "\n";

    // 3. 遍历
    for (int num : dq) std::cout << num << " ";
    return 0;
}
```

### 3. `std::list`（双向链表）



```c++
#include <list>
#include <iostream>

int main() {
    std::list<int> lst = {1, 2, 3};

    // 1. 任意位置插入/删除
    auto it = lst.begin();
    ++it; // 指向第二个元素
    lst.insert(it, 10); // 在2前插入10
    lst.erase(it);      // 删除2

    // 2. 链表特有操作
    lst.sort();       // 排序
    lst.reverse();    // 反转

    // 3. 遍历（不支持随机访问，需迭代器）
    for (int num : lst) std::cout << num << " ";
    return 0;
}
```

### 4. `std::forward_list`（单向链表，C++11）



```c++
#include <forward_list>
#include <iostream>

int main() {
    std::forward_list<int> flst = {1, 2, 3};

    // 1. 单向操作（仅支持insert_after/erase_after）
    auto it = flst.begin();
    flst.insert_after(it, 10); // 在1后插入10
    flst.erase_after(it);      // 删除10

    // 2. 遍历
    for (int num : flst) std::cout << num << " ";
    return 0;
}
```

### 5. `std::array`（固定大小数组，C++11）





```c++
#include <array>
#include <iostream>

int main() {
    // 1. 构造（大小必须是编译期常量）
    std::array<int, 5> arr = {1, 2, 3}; // 剩余元素默认初始化为0

    // 2. 访问
    std::cout << arr[0] << " " << arr.at(1) << "\n";

    // 3. 填充/交换
    arr.fill(10); // 所有元素设为10

    // 4. 遍历
    for (int num : arr) std::cout << num << " ";
    return 0;
}
```

## 二、有序关联容器（红黑树）

### 1. `std::set`（唯一键集合）





```c++
#include <set>
#include <iostream>

int main() {
    std::set<int> s = {3, 1, 4, 1, 5}; // 自动去重+排序

    // 1. 插入
    s.insert(2);

    // 2. 查找
    auto it = s.find(3);
    if (it != s.end()) std::cout << "找到: " << *it << "\n";

    // 3. 删除
    s.erase(1);

    // 4. 遍历（有序）
    for (int num : s) std::cout << num << " "; // 输出: 2 3 4 5
    return 0;
}
```

### 2. `std::multiset`（可重复键集合）





```c++
#include <set>
#include <iostream>

int main() {
    std::multiset<int> ms = {1, 1, 2, 2, 2};

    // 1. 统计重复元素数量
    std::cout << "2的个数: " << ms.count(2) << "\n";

    // 2. 遍历
    for (int num : ms) std::cout << num << " ";
    return 0;
}
```

### 3. `std::map`（键值对映射）



```c++
#include <map>
#include <iostream>
#include <string>

int main() {
    std::map<int, std::string> m;

    // 1. 插入
    m.insert({1, "one"});
    m[2] = "two"; // operator[]插入（若键不存在则创建）

    // 2. 访问
    std::cout << m[1] << " " << m.at(2) << "\n";

    // 3. 查找
    auto it = m.find(3);
    if (it == m.end()) std::cout << "未找到3\n";

    // 4. 遍历（有序）
    for (const auto& p : m) {
        std::cout << p.first << ": " << p.second << "\n";
    }
    return 0;
}
```

### 4. `std::multimap`（一键多值映射）



```c++
#include <map>
#include <iostream>
#include <string>

int main() {
    std::multimap<int, std::string> mm;
    mm.insert({1, "apple"});
    mm.insert({1, "banana"});
    mm.insert({2, "cat"});

    // 1. 查找键为1的所有元素
    auto range = mm.equal_range(1);
    for (auto it = range.first; it != range.second; ++it) {
        std::cout << it->first << ": " << it->second << "\n";
    }
    return 0;
}
```

## 三、无序关联容器（哈希表，C++11）

### 1. `std::unordered_set`（无序唯一集合）



```c++
#include <unordered_set>
#include <iostream>

int main() {
    std::unordered_set<int> us = {3, 1, 4, 1, 5};

    // 1. 插入/查找
    us.insert(2);
    if (us.count(3)) std::cout << "找到3\n";

    // 2. 哈希表信息
    std::cout << "桶数量: " << us.bucket_count() << "\n";

    // 3. 遍历（无序）
    for (int num : us) std::cout << num << " ";
    return 0;
}
```

### 2. `std::unordered_map`（无序键值对）



```c++
#include <unordered_map>
#include <iostream>
#include <string>

int main() {
    std::unordered_map<int, std::string> um;
    um[1] = "one";
    um[2] = "two";

    // 1. 访问/遍历
    for (const auto& p : um) {
        std::cout << p.first << ": " << p.second << "\n";
    }
    return 0;
}
```

## 四、容器适配器

### 1. `std::stack`（栈，LIFO）





```c++
#include <stack>
#include <iostream>

int main() {
    std::stack<int> stk;

    // 1. 入栈/出栈/访问栈顶
    stk.push(1);
    stk.push(2);
    std::cout << "栈顶: " << stk.top() << "\n"; // 2
    stk.pop(); // 弹出2

    // 2. 容量
    std::cout << "大小: " << stk.size() << " 是否空: " << stk.empty() << "\n";
    return 0;
}
```

### 2. `std::queue`（队列，FIFO）



```c++
#include <queue>
#include <iostream>

int main() {
    std::queue<int> q;

    // 1. 入队/出队/访问首尾
    q.push(1);
    q.push(2);
    std::cout << "队头: " << q.front() << " 队尾: " << q.back() << "\n";
    q.pop(); // 弹出1

    // 2. 容量
    std::cout << "大小: " << q.size() << "\n";
    return 0;
}
```

### 3. `std::priority_queue`（优先级队列，默认大顶堆）



```c++
#include <queue>
#include <iostream>

int main() {
    // 1. 默认大顶堆（最大元素优先）
    std::priority_queue<int> pq;
    pq.push(5);
    pq.push(2);
    pq.push(8);
    std::cout << "大顶堆顶: " << pq.top() << "\n"; // 8
    pq.pop();

    // 2. 小顶堆（最小元素优先）
    std::priority_queue<int, std::vector<int>, std::greater<int>> min_pq;
    min_pq.push(5);
    min_pq.push(2);
    min_pq.push(8);
    std::cout << "小顶堆顶: " << min_pq.top() << "\n"; // 2
    return 0;
}
```

## PS:一些知识点

### 1. 访问成员函数返回的是引用

```c++
vector<int> temp;
auto i=temp.front(); //i知识第一个元素的拷贝
auto& i=temp.front(); 	//i是第一个元素的引用，可以改变容器内的值

```

#### 2.迭代器失效问题

当我们堆容器执行一些操作时，原先指向容器的迭代器可能失效，所以**建议要最小化管理迭代器**，即执行操作后，重新定位迭代器

```c++
int main()
{
    //删除偶数，复制奇数
    vector<int> v1{0,1,2,3,4,5,6,6,7,8,9,};
    auto iter=v1.begin();
    //重要：直接使用end(),不保存end返回的迭代器，因为插入时，尾后迭代器就会失效
    while(iter!=v1.end())
    {
        //这里都重新定位iter
        if(*iter%2==0)
        {
            //删除，返回的是删除元素的下一个迭代器
            iter=v1.erase(iter);
        }
        else
        {
            //复制,插入元素是返回第一个新插入的元素位置
            iter=v1.insert(iter,*iter);
            iter+=2;
        }
    }
    return 0;
}
```

