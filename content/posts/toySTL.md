---
title: ToySTL
published: 2026-04-20T18:52:11+08:00
summary: "基于c++11的玩具STL实现:)"
cover:
  image: https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202604201854405.png
tags: [STL]
categories: '数据结构'
draft: false
lang: ''
---



# 一.STL的六大组件

![](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202603071804628.png)

1. 空间分配器：容器要存数据，allocator为其数据开辟内存
2. 适配器：在原有基础上加以简单改在，比如栈基于队列
3. 容器：存储数据的载体
4. 算法：对容器里的数据进行操作得到结果的过程
5. 迭代器：算法与容器之间的桥梁，让算法可以通过iterator访问容器里的元素
6. 仿函数：作用类似于函数，一种可调用形式



# 二.空间分配器

## 1.引言：为什么要使用allocator

对于传统的面向对象，class里面直接使用delete和new管理内存

但是：

1. 直接`new/delete`会把「内存分配」和「对象构造」绑定，性能极差

   - 假设你写一个`vector`，如果直接用`new T[n]`：

     - 一分配内存就必须调用`n`次默认构造函数，哪怕你还没存数据；
     - 扩容时必须拷贝所有对象，哪怕对象支持移动语义；
     - 无法预分配内存（比如`reserve`），每次`push_back`都要重新分配。

     而`allocator`的设计完美解决了这个问题：

     - 你可以先`allocate(100)`分配 100 个对象的内存，不构造任何对象；
     - 等你调用`push_back`/`emplace_back`时，再用`construct`在预分配的内存上构造对象；
     - 扩容时先分配新内存，再用`construct`移动对象，最后`destroy`旧对象、`deallocate`旧内存，全程无多余构造 / 析构。
     - 直接`new/delete`无法实现「自定义内存策略」

2. 直接`new/delete`无法统一所有容器的内存接口

   STL 有 10 + 种容器（`vector`/`list`/`map`/`unordered_map`等），如果每个容器都自己实现内存管理，会导致接口混乱、无法复用。

   而`allocator`提供了**统一的内存接口**，所有容器都用`allocate`/`deallocate`/`construct`/`destroy`这一套函数，不管容器是什么类型，内存管理的逻辑是一致的。 只要接口一致，它就能给所有 STL 容器用。

## 2.如何实现allocator

### 1.功能分析：

1. 分配某种类型的内存，但是不构造对象；释放某地址的内存，不析构对象
2. 在已分配的内存上构造/析构对象
3. 支持任意类型的内存分配

### 2.实现过程中的思考

思路：

要实现allocator基本功能，**注意要明白，暴露的都是接口，肯定有底层的调用实现**

1. 模版类型T，支持为T类型进行内存管理

2. 使用命名空间防止类与标准库冲突

3. 函数

   ```c++
       template <class T>
       class allocator
       {
       public:
           //便于维护
           using value_type = T;
           using pointer = T *;
           using const_pointer = const T *;
           using reference = T &;
           using const_reference = const T &;
           using size_type = size_t;
           using difference_type = ptrdiff_t;
   
       public:
           static pointer allocate();
           static pointer allocate(size_type n);
   
           static void deallocate(pointer ptr);
           static void deallocate(pointer ptr, size_type n);
   
           static void construct(pointer ptr);
           static void construct(pointer ptr, const T &value);
           template <class... Args>
           static void construct(pointer ptr, Args &&...args);
   
           static void destroy(pointer ptr);
       };
   ```



### 3.具体分析函数

#### 1.allocate/deallocate

用于分配T类型内存，**调用::operator new (sizeof(T))**,operatro::delete实现分配内存不构造对象，返回指向内存指针

#### 2.construct

用于在已有内存上进行构造对象，**本质上还是调用内存待分配对象的构造函数**

问题：这个construct是为任意对象进行内存上初始化服务的，这就有一个问题：**我们不知道对象构造函数有几个参数，0,1,2,3······**，可能是好多，我们不可能一个一个重载construct根据参数多少来调用对象的不同构造函数

解决：c++11新特性

✅ 可变参数模板

✅ 万能引用（`Args&&`）

✅ 引用折叠（底层原理）

✅ 完美转发（`forward`）

见文章：

### 3.destroy

实现对象的析构不释放内存，同样根据传的不同的指针类型，调用同一个代码逻辑，他是一个模板函数

**思考**：

1. 如果传的是int*，对与内存上的int（内置类型）对象，有没有必要析构？不需要
2. 自定义对象（string）需要

**性能优化：**

如果我们不管是内置普通类型还是自定义类型都析构，好像有点不至于：），所以思考：

1. 给我任意一个可以指向内存的变量，我都要析构内存上的对象----》有一个统一接口

2. 但是内存上根据对象的不同类型可以有两种选择：1.int，不做事情  2.自定义，正常调用析构

3. 如何区分这两种情况------》萃取判断类型

   ```c++
       // 销毁一个对象,根据对象标签选择析构对象的函数
       template <class T>
       void destroy_one(T *ptr, std::true_type)
       {
           // 空函数
       }
   
       template <class T>
       void destroy_one(T *ptr, std::false_type)
       {
           // 执行对象的析构函数
           ptr->~T();
       }
   
       template <class T>
       void destroy(T *ptr)
       {
           destroy_one(ptr, std::is_trivially_destructible<T>{});
       }
   ```

   

   



