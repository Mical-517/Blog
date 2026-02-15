---
title: C++工程实战 String
published: 2026-02-15T19:31:06+08:00
summary: "关于string用法总结"
cover:
  image: https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202602011340604.png
tags: [string]
categories: '现代c++'
draft: false
lang: ''
---

## 一、string 基础认知

`std::string`是 C++ 标准库（STL）提供的字符串类，定义在`<string>`头文件中，本质是**动态字符数组**（基于`char`），自动管理内存（无需手动分配 / 释放），相比 C 风格字符串（`char*`）更安全、易用，是工程开发的首选。

### 基础准备

使用前必须包含头文件，建议指定命名空间（避免全局命名空间污染）：



```c++
#include <iostream>
#include <string>  // 核心头文件
// 推荐：仅引入string，而非using namespace std
using std::string;
using std::cout;
using std::endl;
```

## 二、string 核心函数（分类讲解 + 示例）

按功能分类讲解高频函数，每个函数附简洁示例，方便快速套用。

### 1. 构造函数（创建 string 对象）



|      函数 / 用法      |             说明              |              示例               |
| :-------------------: | :---------------------------: | :-----------------------------: |
|      `string()`       |           空字符串            |          `string s1;`           |
| `string(const char*)` |      从 C 风格字符串构造      |     `string s2 = "hello";`      |
|   `string(string&)`   |           拷贝构造            |        `string s3 = s2;`        |
|   `string(n, char)`   |       n 个重复字符构造        | `string s4(5, 'a'); // "aaaaa"` |
| `string(s, pos, len)` | 从 s 的 pos 位置取 len 个字符 | `string s5(s2, 1, 3); // "ell"` |

### 2. 元素访问（读取 / 修改单个字符）



|     函数      |                    说明                    |              示例               |
| :-----------: | :----------------------------------------: | :-----------------------------: |
|  `s[index]`   |    访问索引 index 的字符（无越界检查）     |    `char c1 = s2[0]; // 'h'`    |
| `s.at(index)` |    访问索引 index 的字符（越界抛异常）     |  `char c2 = s2.at(1); // 'e'`   |
|  `s.front()`  |       获取第一个字符（等价于 s [0]）       |     `char c3 = s2.front();`     |
|  `s.back()`   | 获取最后一个字符（等价于 s [s.size ()-1]） |     `char c4 = s2.back();`      |
|  `s.c_str()`  |     转为 C 风格字符串（`const char*`）     | `const char* ptr = s2.c_str();` |

> 工程注意：`c_str()`返回的指针仅在 string 对象未修改时有效，修改 string 后指针可能失效。

### 3. 容量与状态（获取 / 调整字符串大小）



|          函数           |                   说明                   |               示例               |
| :---------------------: | :--------------------------------------: | :------------------------------: |
| `s.size()`/`s.length()` |          获取字符数（两者等价）          |   `int len = s2.size(); // 5`    |
|       `s.empty()`       | 判断是否为空字符串（比 size ()==0 高效） |    `if (s1.empty()) { ... }`     |
|     `s.capacity()`      |    获取当前内存容量（可存储的字符数）    |    `int cap = s2.capacity();`    |
|     `s.reserve(n)`      |    预分配 n 个字符的内存（优化性能）     |        `s2.reserve(100);`        |
|      `s.resize(n)`      |    调整字符串长度为 n（不足补 '\0'）     | `s2.resize(8); // "hello\0\0\0"` |
|       `s.clear()`       |   清空字符串（size=0，capacity 不变）    |          `s2.clear();`           |

### 4. 修改操作（增 / 删 / 改字符串）



|            函数            |               说明               |                 示例                  |
| :------------------------: | :------------------------------: | :-----------------------------------: |
|         `s1 = s2`          |               赋值               |            `s1 = "world";`            |
|      `s.append(str)`       |          尾部拼接字符串          |     `s1.append("!"); // "world!"`     |
|      `s.push_back(c)`      |         尾部添加单个字符         |   `s1.push_back('?'); // "world!?"`   |
|       `s.pop_back()`       |         尾部删除单个字符         |     `s1.pop_back(); // "world!"`      |
|    `s.insert(pos, str)`    |       在 pos 位置插入 str        | `s1.insert(5, "123"); // "world123!"` |
|    `s.erase(pos, len)`     |    从 pos 位置删除 len 个字符    |     `s1.erase(5, 3); // "world!"`     |
| `s.replace(pos, len, str)` | 替换 pos 开始的 len 个字符为 str |  `s1.replace(0, 5, "hi"); // "hi!"`   |
|       `swap(s1, s2)`       |  交换两个 string 的内容（高效）  |            `swap(s1, s2);`            |

### 5. 查找与比较（定位 / 对比字符串）





|        函数         |                             说明                             |                 示例                  |
| :-----------------: | :----------------------------------------------------------: | :-----------------------------------: |
| `s.find(str, pos)`  | 从 pos 开始找 str，返回首次出现的索引（找不到返回`string::npos`） |  `size_t pos = s2.find("ll"); // 2`   |
|   `s.rfind(str)`    |               从尾部找 str，返回最后出现的索引               |  `size_t rpos = s2.rfind("l"); // 3`  |
|  `s.compare(str)`   |      比较字符串（返回 0：相等；<0：s 更小；>0：s 更大）      | `int res = s2.compare("hello"); // 0` |
| `s == / != / < / >` |             重载运算符，等价于 compare（更易用）             |     `if (s2 == "hello") { ... }`      |

> 工程注意：`string::npos`是`size_t`类型（无符号整数），判断查找结果时需用`if (pos == string::npos)`，而非`if (pos < 0)`。

### 6. 子串与类型转换（高频工程场景）





|         函数         |                       说明                       |                    示例                     |
| :------------------: | :----------------------------------------------: | :-----------------------------------------: |
| `s.substr(pos, len)` | 截取从 pos 开始的 len 个字符（len 省略则到末尾） |  `string sub = s2.substr(1, 3); // "ell"`   |
|  `stoi/stol/stoll`   |         字符串转整数 / 长整数 / 超长整数         |       `int num = stoi("123"); // 123`       |
|     `stof/stod`      |          字符串转浮点数 / 双精度浮点数           |     `double d = stod("3.14"); // 3.14`      |
|   `to_string(num)`   |                   数字转字符串                   | `string num_str = to_string(123); // "123"` |

## 三、工程中的实际应用

结合真实开发场景，说明 string 的使用技巧和避坑点。

### 1. 核心优势（对比 C 风格字符串）

- 自动内存管理：无需手动`malloc/free`，避免内存泄漏 / 越界；
- 重载运算符：直接用`+`拼接、`==`比较，代码更简洁；
- 动态扩容：无需提前预估长度，超出容量时自动扩容（默认扩容为原容量的 1.5~2 倍）。

### 2. 性能优化技巧

- **预分配内存**：频繁拼接 / 修改字符串时，先用`reserve()`预分配足够容量，避免多次扩容（扩容会拷贝数据，损耗性能）：

  

  

  

  ```c++
  string log;
  log.reserve(1024); // 预分配1024字节
  log.append("user_id: ");
  log.append(to_string(1001));
  log.append(" | time: 2026-02-15");
  ```

  

- **避免频繁拼接**：循环中用`+=`比多次`+`更高效（`+`会创建临时对象）：

  

  ```c++
  // 低效（每次+创建新对象）
  string s;
  for (int i = 0; i < 1000; i++) {
      s = s + to_string(i);
  }
  // 高效（直接追加）
  string s_opt;
  s_opt.reserve(1000);
  for (int i = 0; i < 1000; i++) {
      s_opt += to_string(i);
  }
  ```

  

### 3. 常见工程场景示例

#### 场景 1：文件路径拼接



```c++
// 拼接目录和文件名，自动处理分隔符
string dir = "/home/user/docs";
string file = "readme.txt";
string path = dir + "/" + file; // "/home/user/docs/readme.txt"
```

#### 场景 2：字符串解析（分割 / 提取关键信息）



```c++
// 解析格式："user:1001,name:张三,age:20"
string info = "user:1001,name:张三,age:20";
// 提取user_id
size_t user_pos = info.find("user:") + 5;
size_t user_end = info.find(",", user_pos);
string user_id = info.substr(user_pos, user_end - user_pos); // "1001"

// 提取name
size_t name_pos = info.find("name:") + 5;
size_t name_end = info.find(",", name_pos);
string name = info.substr(name_pos, name_end - name_pos); // "张三"
```

#### 场景 3：日志拼接（类型转换 + 拼接）



```c++
// 生成结构化日志
int code = 200;
double cost = 0.85;
string log = "request: /api/user | code: " + to_string(code) + " | cost: " + to_string(cost) + "s";
cout << log << endl; // "request: /api/user | code: 200 | cost: 0.850000s"
```

### 4. 常见坑与避坑指南

1. **越界访问**：`[]`不做越界检查，工程中优先用`at()`（抛`out_of_range`异常，便于调试）；
2. **c_str () 有效期**：修改 string 后，`c_str()`返回的指针可能失效，需重新获取；
3. **空字符串处理**：判断空字符串优先用`empty()`（比`size()==0`更高效）；
4. **find 返回值**：`find()`找不到时返回`string::npos`（值为`-1`，但类型是`size_t`（无符号），不能用`<0`判断）。

## 总结

### 关键点回顾

1. **核心优势**：`std::string`自动管理内存，重载运算符易用，是工程中替代`char*`的首选；
2. **高频函数**：构造（`string()`/`string(n, char)`）、访问（`at()`/`c_str()`）、修改（`append()`/`replace()`）、查找（`find()`）、转换（`stoi()`/`to_string()`）是开发中最常用的函数；
3. **工程技巧**：频繁修改字符串时用`reserve()`预分配内存，避免循环中多次用`+`拼接，判断空字符串用`empty()`，查找结果判断用`== string::npos`。