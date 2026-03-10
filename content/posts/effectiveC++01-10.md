---
title: EffectiveC++01 10
published: 2026-03-10T23:36:07+08:00
summary: "介绍effectiveC++中的01-10条款"
cover:
  image: https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202603102337548.png
tags: [effectiveC++]
categories: 'effectiveC++'
draft: false
lang: ''
---



# 前言以及一些术语

本书目的是要强调那些常常被漠视的 C++ 编程方向与观点。其他书籍描述 C++语言的各个成分，本书告诉你如何结合那些成分以便最终获得有效程序。其他书籍告诉你如何让程序通过编译，本书告诉你如何回避编译器难以显露的问题。

## 术语

1. 声明式

   所谓声明式(declaration)是告诉编译器某个东西的名称和类型(type)，但略去细节。下面都是声明式：

   ```c++
   extern int x; //对象(object)声明式
   
   std::size_t numDigits(int number); //函数(function)声明式
   
   classwidget; //类(class)声明式
   
   template //模板(template)声明式
   
   class GraphNode; //”typename”的使用见条款42
   ```

   在声明函数时就揭示它的签名式，也及时返回值以及参数的类型，就是一个函数的签名等同于这个函数的类型

   

2. 定义式

   定义式(definition)的任务是提供编译器一些声明式所遗漏的细节。对对象而言，定义式是编译器为此对象拨发内存的地点。对function或function template而言，定义式提供了代码本体。对class或class template而言，定义式列出它们的成员

   

3. 初始化

   初始化(Initialization)是“给予对象初值”的过程。对用户自定义类型的对象而言，初始化由**构造函数**执行。所谓d**efault构造函数是一个可被调用而不带任何实参者。这样的构造函数要不没有参数，要不就是每个参数都有缺省值**：

   ```c++
   class A
   {
   public:
       A(); // default构造函数
   };
   class B
   {
   public:
       explicit B(int x = 0, bool b = true); // default构造函数;
   }; // 关于"explicit", 见以下信息
   ```

   **构造函数被声明为explicit，就可以防止参数发生隐式类型转换**，通常我们也不希望传递的参数发生隐式类型转换，加**入explicit是一个好的习惯**，但是可以手动显示转换参数进行传递

## 命名习惯

帕斯卡驼峰（PascalCase）：首字母大写，用于**类 / 结构体 / 枚举 / 命名空间下的对外类型**

小驼峰（camelCase）：首字母小写，用于**函数 / 成员变量 / 局部变量**

全大写蛇形（SNAKE_CASE）：用于**常量 / 宏定义**

小写蛇形（snake_case）：用于**命名空间**（可选，也可用小驼峰）



# 条款01：将c++视作一个联邦语言

今天的C++已经是个多重范型编程语言(multiparadigm programming language),一个同时支持过程形式(procedural)、面向对象形式(object-oriented)、函数形式(functional)、泛型形式(generic)、元编程形式(metaprogramming)的语言。这些能力和弹性使C++成为一个无可匹敌的工具，但也可能引发某些迷惑：所有“适当用法”似乎都有例外。我们该如何理解这样一个语言呢？

最简单的方法是将C++视为一个由相关语言组成的联邦而非单一语言。在其某个次语言(sublanguage)中，各种守则与通例都倾向简单、直观易懂、并且容易记住。然而当你从一个次语言移往另一个次语言，守则可能改变。为了理解C++，你必须认识其主要的次语言。幸运的是总共只有四个：



1. C.说到底C++仍是以C为基础。区块(blocks)、语句(statements)、预处理器(preprocessor)、内置数据类型(built-in data types)、数组(arrays)、指针(pointers)等统统来自C。许多时候C++对问题的解法其实不过就是较高级的C解法(例如条款2谈到预处理器之外的另一选择，条款13谈到以对象管理资源)，但当你以C++内的C成分工作时，高效编程守则映照出C语言的局限：没有模板(template s),没有异常(exceptions),没有重载(overloading)……

2. Object-Oriented C++。这部分也就是C with Classes所诉求的：classes(包括构造函数和析构函数)，封装(encapsulation)、继承(inheritance)、多态(polymorphism)、virtual函数(动态绑定)……等等。这一部分是面向对象设计之古典守则在C++上的最直接实施。

3. Template C++。这是C++的泛型编程(generic programming)部分，也是大多数程序员经验最少的部分。Template相关考虑与设计已经弥漫整个C++，良好编程守则中“惟template适用”的特殊条款并不罕见(例如条款46谈到调用template functions时如何协助类型转换)。实际上由于template s威力强大，它们带来崭新的编程范型(programming paradigm)，也就是所谓的template metaprogramming(TMP，模板元编程)。条款48对此提供了一份概述，但除非你是template激进团队的中坚骨干，大可不必太担心这些。TMP相关规则很少与C++主流编程互相影响。

4. STL。STL是个template程序库，看名称也知道，但它是非常特殊的一个。它对容器(containers)、迭代器(iterators)、算法(algorithms)以及函数对象(function objects)的规约有极佳的紧密配合与协调，然而template s及程序库也可以其他想法建置出来。STL有自己特殊的办事方式，当你伙同STL一起工作，你必须遵守它的规约。

每个次语言规则都可以转换：

**例如对内置(也就是C-like)类型而言pass-by-value通常比pass-by-reference高效，但当你从C part of C++移往Object-Oriented C++，由于用户自定义(user-defined)构造函数和析构函数的存在，pass-by-reference-to-const往往更好。**





