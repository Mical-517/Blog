---
title: EasyMemoryPool
published: 2026-04-23T23:08:23+08:00
summary: "构建二级空间配置器过程中思考MemoryPool的一些简单实现思路"
cover:
  image: https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202604201935043.png
tags: [MemoryPool]
categories: '思考&&理解'
draft: false
lang: ''
---



# 一.引言以及思路：

原因：MemoryPool就是解决频繁地小内存申请而造成的系统性能损耗，为此对与频繁的小内存申请可以通过一个专门的方法或者控制器来管理

思路：（**参考os的虚拟化内存章节）**

要想减少小内存的频繁想系统申请内存，**我们可以提前申请一块大的内存（MemroyPool），这样如果进程要申请小内存，就只是用这MemoryPool里的内存就好，不用想os申请**，那么我们如何**高效**管理这一大块内存，为了减少碎片，**内存的分配需要时一块一块的，这样每次无论是否达到每块的大小，都分配一块，这样可以有效减少外部碎片的产生**，但是对与最大128bits的块与最小1bits的块差距有点大，造成内部碎片，为了减少内部碎片，我们可以我们需要充分利用空间，归根结底内部碎片是因为块大小太大了，所以对与待存储的不同大小的数据，我们要设计不同的块大小

基于以上思考：我们设计从8,16,32，·····，128大小的块，每张类型的块都由一种列表（**freelist**）管理，freelist包含这些空闲块，在必要的时候（进程申请内存），从**合适**的freelist中找到对应大小的块给进程



大致流程：

1. 进程申请nbits
   1. 找到对应的freelist，然后看这个列表有没有空闲块
      1. 如果有：返回这个空闲块的地址，将空闲块从freelist删除
      2. 如果没有：从MemoryPool分配对应数量的块给这个freelist，然后返回空闲块
2. MemoryPool分配内存给空的freelist：
   1. 先看MemoryPool剩余容量是否够freelist向MemoryPool申请的总量大小
      1. 如果够：返回开始地址与分配的块个数，更新MemoryPool的start
      2. 如果不够：看剩余容量是否够一容量为n的块，如果够就直接讲这个块作为返回值，更新start
      3. 如果剩余容量都不够一块的，那么Memory就需要向os申请一个更大的内存供后续使用
         1. 如果申请成功：现在MemoryPool有内存了，重复调用自身给空的freelist内存
         2. 如果申请失败
            1. 可以看看之前MemoryPool分配给块大小比n更大的freelist是否有空闲块
               1. 如果有，从freelist收回内存给然后重复调用自身给发出请求的进程内存
               2. 如果没有，就只能发出异常了



# 二. 代码实现与理解

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
```

理解：

1. 这里空间分配尤其要注意指向内存的指针大多都是**void***，因为这块内存是划出去给对象的，但是对象什么类型我们不知道，只有调用这个allcator的上层函数知道，所以上层强转就行了。

2. 对与释放ptr指向的内存，就是归还到对应大小的freelist中，就是成为其中的一个节点，所以为了使用节点中的成员，**可以直接将ptr强转为obj***,这样就可以把这段内存加入到对应freelist的头部

3. **理解：各个freelist中的内存块都是从MemoryPool中切出来的，那么对与内存块他想要在freelist中链接起来就需要指针来串联起来，但是如果内存块里存放指针是不是就把挤占了原本规定的内存块的空间，是不是实际内存块的大小比规定的要大**，答案是不会，因为我们节点使用的**union联合体**，同一时间，只有一个成员变量只用内存发挥作用，换句话说：**这个指针不单独占空间，和用户数据共享内存**

    ① 块处于【空闲状态】（在链表中，未分配给用户）

   内存块 → 被当作 `obj` 联合体使用，**`freelist_link` 指针生效**

   - 指针存的是：**下一个空闲块的地址**
   - 作用：串联链表  

   

   ② 块处于【使用状态】（分配给用户）

   内存块 → 被当作用户数据使用，**`data` 生效，指针消失**

   - 整块内存都给用户用
   - 链表指针**不复存在**，没有任何残留

