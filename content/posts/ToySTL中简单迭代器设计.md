---
title: ToySTL中简单迭代器设计
published: 2026-04-25T23:06:17+08:00
summary: "简答介绍STL中迭代器的设计以及关键的萃取技术"
cover:
  image: https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202604201935043.png
tags: [迭代器]
categories: '数据结构'
draft: false
lang: ''
---



# 一.引言：介绍迭代器

迭代器就是一种泛化的指针，泛型算法统一接受迭代器，就可以通过迭代器实现对不同元素的操作，并且对不通过的迭代器设计统一的接口，比如operator*，operator++，泛型算法就可以直接使用这些接口，不用管这些迭代器具体是哪一个容器的迭代器

在使用标准STL中的迭代器可以看到

```c++
vector<int>::iterator 
```

可以看出迭代器是基于容器之下的，**就是说每一种容器我们都要重新设计一个迭代器**，思考：这样也是合理的，毕竟每一种容器底层实现不一样，而迭代器是一种泛化的指针他直接操作容器的元素，所以说根据容器是连续性还是关联型，元素的存放位置就不一样，迭代器的++实现也就不同



# 二.引入萃取的概念

泛型算法取得一个迭代器，不可避免的要用到迭代器执行的元素的类型T

一方面对与算法中普通使用的变量，我们可以直接使用模版函数的参数推导直接使用T，**但是如果我们想要器返回值想要适配迭代器的那个类型怎么办**

举例：

```c++
//泛型算法中distance,返回迭代器距离
//假如说对与vector中的迭代器，返回值想要是size_t
//list								ptrdiff_t
```

既然这个分歧是由于传入的迭代器，那么就在每个迭代器实现内部**声明一个内嵌类型，然后对与所有的迭代器，统一这个声明的名字**

```c++
class iterator1
{
    using size_t fanhuizhi
};
class iterator2
{
    using ptrdiff_t fanhuizhi
}

template<class Iterator>
Iterator::fanhuizhi  disantance();//这样返回值根据传入参数自动判断
```



**问题：标准STL中认为，普通指针也属于迭代器**，显示普通指针显然不能指明内嵌类型，怎么解决：

目的：无论是迭代器还是指针，我们想要的都是他指向的元素类型（**它是迭代器特性中的一个，后续介绍迭代器的特性**），更泛型的说，我们想要迭代器的特性

解决：设计一个类型，通过传入的迭代器，来提取迭代器的类型，这一个过程就是**特性萃取**



# 三.设计思路

### 第一步：发现痛点（为什么直接取不好用？）

**目标**：我想写一个完全通用的通用算法，比如算距离。
**初稿雏形**：

```c++
template <class Iter>
typename Iter::difference_type my_distance(Iter first, Iter last) { ... }
```

**遇到撞墙**：
如果你传入 `vector<int>::iterator`，完美运行，因为类可以有内嵌类型 `difference_type`。
如果你传入原生指针 `int* p1, p2` （原生指针在 C++ 里天生就是一种迭代器），调用 `my_distance(p1, p2)` 时，编译器会试图去寻找 `int*::difference_type`。
**致命报错！C++ 语法根本不允许指针有内嵌类型！**

### 第二步：引入“中间商”（Traits 模式的诞生）

**思考**：既然不能直接问迭代器本身（因为指针不会说话），那我能不能建立一个**信息问询台（中间商）**？
**设计类：`iterator_traits`（泛化与特化）**

- **泛化版（针对正规军）**：如果是普通的类迭代器，中间商直接帮你去问它。

  ```c++
  template <class Iter> struct iterator_traits {
      typedef typename Iter::value_type value_type; 
  };
  ```

  

- **偏特化版（针对特殊群体）**：如果是原生指针 `T*`，中间商帮你“翻译”。

  ```c++
  template <class T> struct iterator_traits<T*> {
      typedef T value_type; // 编译器看到指针，自动提纯出 T
  };
  ```

  

**阶段性成果**：好极了！现在不管是 `vector::iterator` 还是 `int*`，只要放进 `iterator_traits<...>::value_type` 里，都能正确提取属性了！

------

### 第三步：暴露工程缺陷（如何面对“猪队友”？）

**目标**：作为一个库的作者，我们要“防雷”。
**遇到撞墙**：
如果用户手滑，写了一句 `iterator_traits<int>::value_type`。
编译器去匹配泛化版本，试图寻找 `int::value_type`。又是一次**毁灭性的编译报错**。而且这种报错通常连带几百行模板调用栈，让用户崩溃。

**思考**：我必须在中间商干活之前，**加一道“安检门”**。如果传进来的根本不是迭代器，中间商应该闭嘴（返回空结构体），而不是强行提取报错。

### 第四步：打造“雷达”探测器（SFINAE 的应用）

**目标**：在不报错的前提下，悄悄探测体类 `T` 到底有没有 `iterator_category` （这是作为迭代器的资格证）。
**设计类：`has_iterator_cat`**

- **思路**：利用 C++ 函数重载。我造两个测试函数，一个专门接收带 `iterator_category` 的（返回类型大小为 1），一个专门当垃圾桶接收其他的（返回类型大小为 2）。

- 根据 `sizeof` 测试的结果得出 `true` 还是 `false`。

  ```c++
  template <class T> struct has_iterator_cat {
      static const bool value = ...; // 探测结果
  };
  ```

  

------

### 第五步：搭建“条件装配线”（编译期 if-else）

**目标**：探测结果有了，但怎么把探测结果（bool）跟中间商（`iterator_traits`）结合起来呢？
**设计类：`iterator_traits_impl`**

- **思路**：用另一个模板类做分支。如果雷达说是 `true`，我才把里面的 `typedef` 吐出来；如果是 `false`，我就什么都不写。

  ```c++
  template <class Iter, bool> struct iterator_traits_impl {}; // 默认空（针对 false）
  
  template <class Iter> struct iterator_traits_impl<Iter, true> {
      typedef typename Iter::value_type value_type; // 只有 true 才会编译这一段
  };
  ```

  

### 第六步：严格的“资质审查”（二次防线）

**目标**：就算一个类碰巧有个 `category`，那它也未必是 STL 承认的迭代器分类。
**设计类：`iterator_traits_helper`**

- **思路**：要求它不仅有 `category`，而且这个 `category` 必须能转换成 STL 规定的那 5 种 Tag 之一（比如 `input_iterator_tag` 或 `output_iterator_tag`）。

  ```c++
  template <class Iter> struct iterator_traits_helper<Iter, true>
      : public iterator_traits_impl<Iter, 
        std::is_convertible<...>::value> // 再次求值验证
  {};
  ```

  

### 第七步：最终的封装（对用户隐藏复杂性）

**目标**：把上面这堆雷达、安检门、装配线，全部包进一个整洁的盒子里给用户用。
**最终定稿：完善后的 `iterator_traits`**

```c++
template <class Iterator>
struct iterator_traits 
  : public iterator_traits_helper<Iterator, has_iterator_cat<Iterator>::value> {};
```



- **用户视角**：用起来依然和第一步一样爽：`iterator_traits<T>::value_type`。
- **底层视角**：一旦 `T` 是垃圾类型，雷达 `has` 报 false，传给 `helper`，`helper` 匹配不到 true 的版本，退化为空；由于最终的 `traits` 是空的，它直接引发一个“没有该成员”的温和错误，或者在 SFINAE 机制下直接让整个函数重载静默失效（这正是一个健壮的 C++ 库所期望的）。



### 总结：

上面的思考流程如果转换为运行过程中的if-else语句就是

```c++
if(传入的迭代器内部确实有迭代类型标签)
    if(这个标签的类型确实是STL规定的5个标签之一)
        中间商iterator_traits就可以直接使用这个迭代器内部的内嵌类型
    else
        iteraor_tarits就是空的
else
    iterator_traits
    
```


