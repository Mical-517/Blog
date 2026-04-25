---
title: 模版元编程(实现iterator)
published: 2026-04-25T23:41:08+08:00
summary: "实现ToySTL中学习关于模版元编程思想"
cover:
  image: https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202604252344661.png
tags: [模版元编程]
categories: '思考&&理解'
draft: false
lang: ''
---



# 🌟 一个核心哲学：时空转移（将计算从运行时提前到编译期）

在传统的 C++ 编程中，程序的逻辑是：**“跑起来之后，CPU 再决定怎么做”**。
而在模板元编程中，它的终极哲学是：**“让编译器在生成机器码之前，就把一切逻辑算好、把路铺好”**。

它改变了我们写代码的基础元素：

- **普通编程**：把 **数值（Value）** 作为变量，把 **函数（Function）** 作为加工机器。
- **元编程**：把 **类型（Type）** 作为变量，把 **模板（Template）** 作为加工机器。

这也是为什么在 TMP 的世界里，代码具有 **“零运行时开销（Zero Runtime Overhead）”** 和 **“极强类型安全”** 的威力。

------

# 🛠️ 五大基础武器（你在源码中见过的招式）

#### 1. 类型萃取（Type Traits）：打破壁垒的“中间商”

- **思想**：不是所有类型都“会说话”（比如原生指针 `int*` 没有内嵌的 `typedef`）。为了让算法统一处理所有类型，我们需要一个中间层来提取特征。

- **在源码中的体现**：

  ```c++
  type_traits
  ```

  

- **模式套路**：通过泛化版本处理正规的自定义类，通过偏特化版本

  ```c++
  ::value_type
  ```

  

- 处理特殊的原生元素，从而向外界暴露绝对统一的接口（如 [::value_type](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/10c8e557c8/resources/app/out/vs/code/electron-browser/workbench/workbench.html)）。

#### 2. 元函数（Metafunction）：没有返回语句的函数

- **思想**：由于要在编译期运算，我们不能用普通的 `return`。元函数的“参数”放在尖括号 `<...>` 里，“返回值”则固定约定为里面的静态常量 [::value](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/10c8e557c8/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 或者类型别名 `::type`。
- **在源码中的体现**：`m_integral_constant`、`is_pair`
- **模式套路**：巧妙利用类型的继承（如继承 `m_false_type`）来“白嫖”父类的 [value = false](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/10c8e557c8/resources/app/out/vs/code/electron-browser/workbench/workbench.html)，实现逻辑判断基石。

#### 3. SFINAE（替换失败并非错误）：无伤嗅探特工

- **思想**：当你想知道一个未知类型 [T](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/10c8e557c8/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 是否具备某种特征（例如有没有 [iterator_category](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/10c8e557c8/resources/app/out/vs/code/electron-browser/workbench/workbench.html)）时，直接访问往往会导致编译爆炸。SFINAE 提供了一种“试错但不报错”的机制。
- **在源码中的体现**：[has_iterator_cat](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/10c8e557c8/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 里面的两组 [test](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/10c8e557c8/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 函数重载。
- **模式套路**：利用 **不求值上下文（`sizeof`）** + **函数重载优先级**。故意写一个精准匹配的测试函数和一个用可变参数 `...` 兜底的垃圾桶函数。如果精准匹配导致类型替换失败，编译器会自动默默选用垃圾桶函数，从而让我们通过 `sizeof` 大小反推出类型是否合格。

#### 4. 布尔派发（Bool Dispatch）：编译期的 `if-else`

- **思想**：如何在不执行代码的情况下做条件分支？答案是利用模板的泛化与偏特化。
- **在源码中的体现**：[iterator_traits_helper](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/10c8e557c8/resources/app/out/vs/code/electron-browser/workbench/workbench.html)
- **模式套路**：
  - **充当 `else`**：写一个带有 `bool` 参数的泛化模板（里面留空，防止瞎提取报错）。
  - **充当 `if(true)`**：写一个偏特化模版，在里面放入真实的执行逻辑。
  - **入口调用**：将前面利用 SFINAE 算出的 [bool::value](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/10c8e557c8/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 塞入这个模板，编译器会自动选择去留。

#### 5. 标签分发（Tag Dispatch）：实现重载路由

- **思想**：有些算法对于不同的数据结构有快慢之分（例如 `i++` 对比 `i+=n`）。我们需要根据类型的不同，在编译期智能路由到最快的算法，且对外仅提供一个统一的接口名。
- **在源码中的体现**：5种空的 `iterator_tag` 和 `distance_dispatch()` / `advance_dispatch()`。
- **模式套路**：定义一系列**有继承关系的空结构体**作为标签（利用继承实现类型的向下兼容，因为 [is_convertible](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/10c8e557c8/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 支持派生类转基类），然后在函数调用的最后一个参数传入 `Tag()` 原型。编译器会根据 Tag 的类型自动重载到最优的 `_dispatch` 函数体。



# 根据具体代码解释上面的基础技术

## 1. 类型萃取（Type Traits）：打破壁垒的“中间商”

就是解决对与普通指针，不能够通过内嵌类型得到这个指针迭代器的特性，**通过iterator_traits的特化版本和偏特化版本实现根据传入的是指针还是其他类型实例化不同的iterator_traits类**

```c++
// 定义type_traits类,就是调用上面的if-else对应的类
    template <class Iterator>
    class iterator_traits : public iterator_traits_helper<Iterator, has_iterator_cat<Iterator>::value>
    {
    };

    // 偏特化，适配迭代器是*，const*
    template <class T>
    class iterator_traits<T *>
    {
    public:
        using iterator_category = randon_access_interator_tag;
        using value_type = T;
        using pointer = T *;
        using reference = T &;
        using difference_type = ptrdiff_t;
    };

    template <class T>
    class iterator_traits<const T *>
    {
    public:
        using iterator_category = randon_access_interator_tag;
        using difference_type = ptrdiff_t;
        using value_type = const T;
        using pointer = const T *;
        using reference = const T &;
    };
```



## 2. 元函数（Metafunction）：没有返回语句的函数

**在模版元编程中，一个我们可以将一个模版类视为一个元函数，他实现了类似函数的功能，接受一个模版类型参数（不同点1：普通函数接受一个值），没有return最为返回值，返回值累内部定义的一个静态变量（第二个不同点）**

**思想**：由于要在编译期运算，我们不能用普通的 `return`。元函数的“参数”放在尖括号 `<...>` 里，“返回值”则固定约定为里面的静态常量 [::value](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/10c8e557c8/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 或者类型别名 `::type`

```c++
    template <class Iterator>
    struct has_iterator_cat
    {
    private:
        // 根据Iteratro是否内嵌了类型来定义两个函数，根据返回值大小判断是否有内嵌类型
        struct two
        {
            char a;
            char b;
        };
        template <class U>
        static two test(...);
        template <class U>
        static char test(typename U::iterator_category * = nullptr);

    public:
        static const bool value = (sizeof(test<Iterator>(nullptr)) == sizeof(char));
    };
```

比如这个模板类，类比函数它的功能就是：接受一个迭代器类型（模版参数），判断迭代器中是否有迭代器类型标签（通过访问value的值是true还是false可以判断）

```c++
namespace mystl
{
    //这是一个具有包装作用的模板类（元函数），他可以根据传入的类型以及具体的值，保存这个类型以及这个值，后续
    //可以直接使用这个类来访问内部成员
    template <class T, T v>
    class m_integral_constant
    {
        static const T value = v;
    };

    // 给包装bool值的这个固化器器别名
    template <bool b>
    using m_bool_constant = m_integral_constant<bool, b>;

    // 定义true和false类型，可以根据类来直接得到false或者是type，便于后续iterator中使用value返回统一
    using m_true_type = m_bool_constant<true>;
    using m_false_type = m_bool_constant<false>;
}
```

