---
title: MemoryPool简单实现Allocator
published: 2026-04-23T21:04:30+08:00
summary: "关于ToyStl中空间配置器的扩展"
cover:
  image: https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202604201854405.png
tags: [STL]
categories: '数据结构'
draft: false
lang: ''
---



# 一.可优化点

基于前面stl的要求：内存精细化控制，分配与构造解耦，所以我们要手动实现一个空间配置器然后让所有容器统一使用空间配置器进行内存的管理，内存上对象的构造与析构

但是如果只是对`::operator new`,`::new`的简单的一层封装，虽然可以实现构造与分配内存解耦的问题，但是性能不好：如果有多个小内存的申请，每一次申请都要调用`::operator new`,这显然消耗系统资源，所以针对申请小内存的请求我们应该有一种allocator控制，大内存有一种allocator控制，然后将这两种allocator在进行最后一层抽象为Allocator，这样容器都已Allocator为借口，然后调用底层的一，二级空间配置器进行内存分配

总结：

空间配置器是 STL 所有容器的底层内存基石，它将**内存分配 / 释放**与**对象构造 / 析构**完全解耦，核心解决三大问题：

1. **职责解耦**：把内存管理逻辑从容器中抽离，让容器专注于数据结构与业务逻辑，符合 C++ 泛型编程的设计思想；
2. **效率优化**：解决 C++ 原生`new/delete`对小内存分配的高频系统调用开销、以及内存碎片化问题；
3. **安全保证**：提供异常安全的内存操作语义，保证批量内存初始化的「commit or rollback」（要么全成功，要么全回滚），同时兼容 C++ 的内存不足（OOM）处理机制。

# 二. 空间配置器的实现



## 1.功能分析

### GI STL 两级空间配置器的核心设计

SGI STL 并未使用上述简易 allocator，而是设计了**两级分工的空间配置器**，这是文档的核心重点：

1. **一级配置器**：处理**大于 128 字节的大内存分配**，直接封装系统级内存管理函数，同时模拟 C++ 的`set_new_handler`机制，处理 OOM 场景；
2. **二级配置器**：处理**小于等于 128 字节的小内存分配**，通过「内存池 + 自由链表（free lists）」的次配置机制，避免频繁调用系统内存函数的开销，同时解决小内存碎片化问题；
3. **统一封装层**：通过`simple_alloc`模板对两级配置器做包装，把内存分配单位从「字节」转换为「对应类型的元素大小」，让容器无需关心底层是一级还是二级配置器，完全兼容 STL 标准接口。

### rebind机制

rebind机制在于让配置器更灵活的为容器配置内存，比如`vector<int,allocator<int>> v1`这里的配置器是默认参数为int，所以默认vector使用配置器分配存储元素为int类型的内存，但是**vector里面如果存储的是node<int>**,此时没有rebind，allocator就只能分配int的内存，所以**rebind就是在嵌套一个allocator<U>结构体，这样，如果allocator要为容器分配的不是基础类型，那么就可以让使用allocator<int>::allocator<node<int>>**,这个成员与allocator<node<int>>,完全一样，就是类自己在自己本身上在套上一个自己的成员本身。

## 2.一级空间配置器实现

### 1.分析

**底层实现**：SGI 原版基于`malloc/free/realloc`实现，核心原因是历史兼容与`realloc`的内存重分配能力；

**OOM 异常处理**：

- 当内存分配失败时，不会直接终止程序，而是循环调用用户注册的「内存不足处理函数」，尝试释放内存后再次分配；
- 模拟 C++ 的`set_new_handler`机制，提供接口让用户自定义 OOM 处理逻辑；若用户未注册处理函数，则直接抛出内存异常或终止程序；

**无类型绑定**：不使用模板类型参数，保证所有类型的一级配置器共享同一套内存管理逻辑，避免模板实例化带来的代码冗余

### 2.代码以及先关知识点

```c++
    /**
     * @brief 一级空间配置器
     * 职责：管理大内存分配
     * 功能：
     * 1.内存分配与释放
     * 2.内存分配不足时可以调用oom（out of memory）函数
     */
    template <int inst>
    class Allocator_level1_template
    {
    private:
        /**
         * @brief 函数指针变量，用来指向后续的oom函数，注意静态成员变量，每个类一份
         */
        static void (*Allocator_level1_oom_hander)();
        static void *oom_alloc(size_t n);

    public:
        /**
         * @brief 分配内存（接口）
         * @return void* 返回指向分配内存的指针
         * @param n 分配内存的大小
         */
        static void *allocate(size_t n)
        {
            void *result = ::operator new(n, std::nothrow);
            if (result == nullptr)
            {
                return oom_alloc(n);
            }
            return result;
        }
        /**
         * @brief 释放内存
         * @param ptr,size_t,ptr是指向内存指针，size_t只是占位统一接口
         */
        static void deallocate(void *ptr, size_t)
        {
            if (ptr == nullptr)
                return;
            ::operator delete(ptr);
        }

        /**
         * @brief 用于指定类中静态函数指针的指向
         * 类似于c++中的set_new_hander
         *
         * @param f 是一个函数指针，就收void返回void
         *
         * @return 返回一个原来的函数指针，接受void，返回void
         *
         */

        void(*set_Allocator_levle1_oom_hander(void (*f)()))
        {
            void (*old)() = Allocator_level1_oom_hander;
            Allocator_level1_oom_hander = f;
            return old;
        }
    };

    /**
     * @brief 实现oom_alloc,用于当无法分配内存时候调用指针指向的处理机制函数
     * 一个无限循环，调用之前指向的处理函数
     */
    template <int inst>
    void *Allocator_level1_template<inst>::oom_alloc(size_t n)
    {
        void *result;
        void (*my_hander)() = Allocator_level1_oom_hander;
        while (true)
        {
            if (my_hander == nullptr)
            {
                throw std::bad_alloc();
            }
            (*my_hander)();
            result = ::operator new(n);
            if (result != nullptr)
            {
                return result;
            }
        }
    }

    // 初始化类中静态成员
    template <int inst>
    void (*Allocator_level1_template<inst>::Allocator_level1_oom_hander)() = nullptr;
```

知识点：

1. 关于set_new_hander机制，如何实现的以及关于函数指针作为返回值以及参数的形式认识

   ```c++
   void(*set_Allocator_levle1_oom_hander(void (*f)()))
           {
               void (*old)() = Allocator_level1_oom_hander;
               Allocator_level1_oom_hander = f;
               return old;
           }
   //这是一个函数，名字是set_Allocator_levle1_oom_hander，接受一个void(void)的函数指针f作为参数，返回一个
   //void(void)函数指针。定义一个函数指针变量old
   ```

2. 关于oom函数的实现与理解



## 二级Allocator的实现

### 1.分析

为了减少memory的频繁申请，所以二级空间配置器提前申请一批大内存，然后当进程要申请内存使用的时候，Memorypool就从已经申请好的memory里面借内存给进程使用，所以我们要有一种数据结构管理memorypool，这就是**freelist**，设计如下：

**8 字节对齐规则**：将任意小内存需求向上对齐到 8 的整数倍（如 30 字节→32 字节），最终维护 16 个固定大小的内存块规格，减少自由链表的管理成本；**尽量减少**了因为申请内存不一样而造成的内存碎片化以及**尽量适配**了不同0~128内存的需求

**自由链表（free lists）设计**：

- 维护 16 个链表，分别管理 8/16/24/.../128 字节的内存块，每个链表对应固定大小的内存池分片；
- 采用`union obj`实现链表节点，一物二用：既可以作为指向下一个节点的指针，也可以指向实际的内存数据，**完全避免链表指针带来的额外内存开销**；

<img src="https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202604232145569.png" alt="image-20260423214511268" style="zoom: 50%;" />

### 2.实现与一些知识点

```c++
    enum class FREELIST
    {
        _ALIGN = 8,
        _MAX_BITS = 128,
        _NFREELISTS = _MAX_BITS / _ALIGN
    };

    template <int inst>
    class Allocator_level2_template
    {
    private:
        // 使用union定义空闲列表的节点
        union obj
        {
            union obj *freelist_link;
            char *data[1];
        };

        // 定义空闲列表
        static obj *free_list[static_cast<size_t>(FREELIST::_NFREELISTS)];

        // 描述内存池开始地址，结束地址，已经向os申请多少memory
        static char *pool_begin;
        static char *pool_end;
        static size_t heap_size;

        // 工具函数1:将申请内存与8及8的倍数对其
        static size_t round_up(size_t n)
        {
            return static_cast<size_t>((n + static_cast<size_t>(FREELIST::_ALIGN) - 1) & ~(static_cast<size_t>(FREELIST::_ALIGN) - 1));
        }

        // 工具函数2：找到管理内存块为round_up(n)的freelist
        static size_t free_list_index(size_t n)
        {
            return (n + static_cast<int>(FREELIST::_ALIGN) - 1) / static_cast<size_t>(FREELIST::_ALIGN) - 1;
        }
        // 工具函数3：当freelist没有内存时，向pool申请对应n的内存给freelist，freelist管理逻辑
        static void *refill(size_t n);
        // 工具函数4：pool分配已有内存给freelist逻辑
        static void *pool_allocate(size_t n, int &nobjs);

    public:
        // 接口：分配小于128bits的memory
        static void *allocate(size_t n)
        {
            if (n >= static_cast<size_t>(FREELIST::_MAX_BITS))
            {
                _level1_allocator::allocate(n);
            }

            // 从管理对应字节大小的freelist分配block of memory
            // 先找到对应管理内存块为round_up(n)的freelist
            obj **my_free_list = free_list + free_list_index(n);
            obj *result = *my_free_list;
            if (result != nullptr)
            {
                *my_free_list = result->freelist_link;
                return result;
            }
            else
            {
                // 从pool一次性分配多个对应的内存块给对应级别的空闲列表
                return refill(round_up(n));
            }
        }

        // 接口：释放指向的内存
        static void deallocate(void *p, size_t n)
        {
            // 将这个memory返回给freelist
            obj *ptr = (obj *)(p);
            if (n >= static_cast<size_t>(FREELIST::_MAX_BITS))
            {
                _level1_allocator::deallocate(p, n);
            }
            obj **my_free_list = free_list + free_list_index(round_up(n));
            ptr->freelist_link = *my_free_list;
            *my_free_list = ptr;
        }
    };

    // 实现refill
    template <int inst>
    void *Allocator_level2_template<inst>::refill(size_t n)
    {
        int nobjs = 20;
        char *ptr = (char *)pool_allocate(n, nobjs);

        obj **my_free_list;
        obj *current;
        obj *next;
        void *result;

        if (nobjs == 1)
        {
            return ptr;
        }
        my_free_list = free_list + free_list_index(round_up(n));
        result = (void *)ptr;
        current = *my_free_list = (obj *)(ptr + n);

        int i = 1;
        while (i != nobjs)
        {
            // 重要
            next = (obj *)((char *)current + n);
            current->freelist_link = next;
            current = next;
            i++;
        }
        current->freelist_link = nullptr;
        return result;
    }

    // 实现pool_allocate
    template <int inst>
    void *Allocator_level2_template<inst>::pool_allocate(size_t n, int &objs)
    {
        char *result = nullptr;
        // 先计算总共需要分配多少已有内存，当前内存池剩余多少内存
        size_t total_need = static_cast<size_t>(n * objs);
        size_t pool_left = static_cast<size_t>(pool_end - pool_begin);

        // 1.如果剩余>=所需
        if (pool_left >= total_need)
        {
            // 更新描述pool的变量
            result = pool_begin;
            pool_begin += total_need;
            return (void *)result;
        }
        // 2.如果剩余<所需，但可以满足至少一个内存块所需
        else
        {
            if (pool_left >= n)
            {
                // 尽可能返回可满足要求size=n的块数，然后更新pool
                objs = (int)(pool_left / n);
                total_need = n * objs;
                char *result = pool_begin;
                pool_begin += total_need;
                return (void *)result;
            }
            // 3.pool_left<n,需要额外向os再次申请一片内存,注意此时如果poo里面还有没有分配给freelist的memory，
            // 就要先挂载到相应的freelist，不能浪费
            else
            {
                // 统计要申请多少bits memory
                size_t apply_memory_num = total_need * 2 + round_up(heap_size >> 4);
                obj **my_free_list = nullptr;
                obj *ptr = nullptr;
                if (pool_left != 0)
                {
                    my_free_list = free_list + free_list_index(round_up(pool_left));
                    ptr = (obj *)(pool_begin);
                    ptr->freelist_link = *my_free_list;
                    *my_free_list = ptr;
                }
                pool_begin = (char *)::operator new(apply_memory_num);
                if (pool_begin == nullptr)
                {
                    // 就要从其他管理更大空闲节点的空闲列表收回来pool原来分配给他的memory
                    for (size_t i = n; i <= (size_t)(FREELIST::_MAX_BITS); i += (size_t)(FREELIST::_ALIGN))
                    {
                        obj **my_free_list = free_list + free_list_index(round_up(i));
                        obj *ptr = *my_free_list;
                        if (ptr != nullptr)
                        {
                            pool_begin = (char *)(ptr);
                            pool_end = pool_begin + i;
                            return pool_allocate(n, objs);
                        }
                    }
                    // 如果还是没有memory，就调用一级空间分配器进行错误处理
                    pool_end = nullptr;
                    pool_begin = (char *)_level1_allocator::allocate(apply_memory_num);
                }

                // pool有memory了，再次调用自己分配
                heap_size += apply_memory_num;
                pool_end = pool_begin + apply_memory_num;
                return pool_allocate(n, objs);
            }
        }
    }
    using _level2_alocator = Allocator_level2_template<0>;

    template <int inst>
    typename Allocator_level2_template<inst>::obj *
        Allocator_level2_template<inst>::free_list[static_cast<size_t>(FREELIST::_NFREELISTS)]{
            nullptr,
            nullptr,
            nullptr,
            nullptr,
            nullptr,
            nullptr,
            nullptr,
            nullptr,
            nullptr,
            nullptr,
            nullptr,
            nullptr,
            nullptr,
            nullptr,
        };

    template <int inst>
    char *Allocator_level2_template<inst>::pool_begin = nullptr;
    template <int inst>
    char *Allocator_level2_template<inst>::pool_end = nullptr;
    template <int inst>
    size_t Allocator_level2_template<inst>::heap_size = 0;

    /**
     * @class Allocator
     * @brief 再次封装一，二级空间配置器，用来规范allocator的接口统一
     * 这里就是为了可以作为容器底层适配器而做的一层抽象
     *
     */

    template <class T>
    class Allocator
    {
    private:
        // 定义类型别名
        using value_type = T;
        using pointer = T *;
        using const_pointer = const T *;
        using reference = T &;
        using const_reference = const T &;
        using size_type = size_t;          // 大小类型，无符号longl long整数，
        using difference_type = ptrdiff_t; // 差值类型，有符号long long整数

        // 使用rebind来适配容器内部所需要的其他类型空间分配器
        template <class U>
        struct rebind
        {
            using other = Allocator<U>;
        };
        // 优化提示类型，因为这里的allocator封装底层一二级配置器，并且他们都是单例的全局静态变量，所以是无状态的配置器
        using propagate_on_container_copy_assignment = std::false_type;
        using propagate_on_container_move_assignment = std::false_type;
        using propagate_on_container_swap = std::false_type;
        using is_always_equal = std::true_type;

    public:
        Allocator() noexcept = default;
        Allocator(const Allocator &) noexcept = default;

        template <class U>
        Allocator(const Allocator<U> &) noexcept {}

        ~Allocator() = default;

        /**
         * @brief 调用底层空间配置器进行内存分配与释放
         * @param n   分配n*sizeof（T）字节
         */

        static pointer allocate(size_type n = 1) noexcept;

        static void deallocate(pointer ptr, size_type n = 1) noexcept;

        static void destroy(pointer ptr) noexcept;

        size_type max_size() const noexcept
        {
            return size_type(-1)/sizeof(T);
        }

        //因为我们的 allocator 是无状态的，所以所有 allocator 都相等
        template<class T,class U>
        bool operator==(const Allocator<T>&,const Allocator<U>&)
        {
            return true;
        }

        template<class T,class U>
        bool operator!=(const Allocator<T>&,const Allocator<U>&)
        {
            return true;
        }

        /**
         * @brief
         * constrct 接口：用于在已分配内存上进行对象的构造与析构，为了适配各种类型参数与个数的构造函数使用
         * 模版构造函数列表，以及为了防止参数在传递过程中改变，使用完美转发机制
         */

        template <class... Args>
        static void construct(pointer ptr, Args &&...args) noexcept;
    };

    template <class T>
    T *Allocator<T>::allocate(size_type n) noexcept
    {
        if (n == 0)
            return nullptr;
        return (T *)mystl::_level2_alocator::allocate(sizeof(T) * n);
    }

    template <class T>
    void Allocator<T>::deallocate(pointer ptr, size_type n) noexcept
    {
        if (ptr == nullptr)
            return;
        mystl::_level2_alocator::deallocate(ptr, sizeof(T) * n);
    }

    template <class T>
    void Allocator<T>::destroy(pointer ptr) noexcept
    {
        if (ptr == nullptr)
            return;
        mystl::destroy(ptr);
    }

    template <class T>
    template <class... Args>
    void Allocator<T>::construct(pointer ptr, Args &&...args) noexcept
    {
        if (ptr == nullptr)
            return;
        mystl::construct(ptr, mystl::forward<Args>(args)...);
    }

};
```



知识点：

1. enumclass，在c++11中，枚举类不能隐式的转换为常量，必须显示的时候static_cast<int>转换，并且访问要加作用域

2. 关于union的使用以及优点

3. 如何描述memorypool的一些基本变量

4. rand_up()，free_list_index()实现思想以及理解功能：
   1. rand_up(data)：可以让data变为距离他最近的2^n比如4->8,15->16,0->16，便于后续内存申请，是如何实现的
   2. free_list_index：可以计算申请内存为n应该找第几个freelist

5. 关于优化提示类型以及**如何看待这个allocator是没有状态的**

   关于allocator这个类是无状态类型的，这是因为这个类是封装底层的空间配置器的，所以，**他没有自己的成员变量，这就意味着所有它的对象拥有的函数都是同一份的，所以allocator之间复制都是没有意义的，即无状态的，所有allocator都是一样的**，基于此，c++11提供了优化提示类型来针对这种这种所有实例都是一样的类提供优化

6. size_t(-1)表示size_t类型能表示的最大值

   无符号数**不能表示负数**，会发生**回绕（wrap around）**

   结果直接变成 **该类型的最大值**











