# valgrind 的使用方法 - 详细手册

## Valgrind 的使用

Valgrind 是一个 GPL 的软件，用于 Linux (For x86, amd64 and ppc32) 程序的内存调试和代码剖析。你可以在它的环境中运行你的程序来监视内存的使用情况，比如 C 语言中的 malloc 和 free 或者 C++ 中的 new 和 delete。使用 Valgrind 的工具包，你可以自动的检测许多内存管理和线程的 bug，避免花费太多的时间在 bug 寻找上，使得你的程序更加稳固。

## Valgrind 的主要功能

Valgrind 工具包包含多个工具，如 Memcheck、Cachegrind、Helgrind、Callgrind、Massif。下面分别介绍各工具的作用：

### Memcheck

工具主要检查下面的程序错误：

- 使用未初始化的内存 (Use of uninitialised memory)
- 使用已经释放的内存 (Reading/writing memory after it has been free’d)
- 使用超过 malloc 分配的内存空间 (Reading/writing off the end of malloc’d blocks)
- 对堆栈的非法访问 (Reading/writing inappropriate areas on the stack)
- 申请的空间是否有释放 (Memory leaks – where pointers to malloc’d blocks are lost forever)
- malloc/free/new/delete 申请和释放内存的匹配 (Mismatched use of malloc/new/new [] vs free/delete/delete [])
- src 和 dst 的重叠 (Overlapping src and dst pointers in memcpy () and related functions)

### Callgrind

Callgrind 收集程序运行时的一些数据，函数调用关系等信息，还可以有选择地进行 cache 模拟。在运行结束时，它会把分析数据写入一个文件，callgrind_annotate 可以把这个文件的内容转化成可读的形式。

### Cachegrind

它模拟 CPU 中的一级缓存 I1、D1 和 L2 二级缓存，能够精确地指出程序中 cache 的丢失和命中。如果需要，它还能够为我们提供 cache 丢失次数、内存引用次数，以及每行代码、每个函数、每个模块、整个程序产生的指令数，这对优化程序有很大的帮助。

### Helgrind

它主要用来检查多线程程序中出现的竞争问题。Helgrind 寻找内存中被多个线程访问，而又没有一贯加锁的区域，这些区域往往是线程之间失去同步的地方，而且会导致难以发掘的错误。Helgrind 实现了名为”Eraser” 的竞争检测算法，并做了进一步改进，减少了报告错误的次数。

### Massif

堆栈分析器，它能测量程序在堆栈中使用了多少内存，告诉我们堆块、堆管理块和栈的大小。Massif 能帮助我们减少内存的使用，在带有虚拟内存的现代系统中，它还能够加速我们程序的运行，减少程序停留在交换区中的几率。

## Valgrind 安装

1. 到[www.valgrind.org](https://www.valgrind.org)下载最新版 valgrind-3.2.3.tar.bz2
2. 解压安装包：`tar –jxvf valgrind-3.2.3.tar.bz2`
3. 解压后生成目录 valgrind-3.2.3
4. 切换目录：`cd valgrind-3.2.3`
5. 配置：`./configure`
6. 编译安装：`make;make install`

## Valgrind 使用

用法：`valgrind [options] prog-and-args`

### 常用选项（适用于所有 Valgrind 工具）

1. `-tool=<name>`：最常用的选项，运行 valgrind 中名为 toolname 的工具，默认 memcheck
2. `-h –help`：显示帮助信息
3. `-version`：显示 valgrind 内核的版本，每个工具都有各自的版本
4. `-q –quiet`：安静地运行，只打印错误信息
5. `-v –verbose`：更详细的信息，增加错误数统计
6. `-trace-children=no|yes`：跟踪子线程？[no]
7. `-track-fds=no|yes`：跟踪打开的文件描述？[no]
8. `-time-stamp=no|yes`：增加时间戳到 LOG 信息？[no]
9. `-log-fd=<number>`：输出 LOG 到描述符文件 [2=stderr]
10. `-log-file=<file>`：将输出的信息写入到 filename.PID 的文件里，PID 是运行程序的进程 ID
11. `-log-file-exactly=<file>`：输出 LOG 信息到 file
12. `-log-file-qualifier=<VAR>`：取得环境变量的值来做为输出信息的文件名。[none]
13. `-log-socket=ipaddr:port`：输出 LOG 到 socket，ipaddr:port

### LOG 信息输出

1. `-xml=yes`：将信息以 xml 格式输出，只有 memcheck 可用
2. `-num-callers=<number>`：show <number> callers in stack traces [12]
3. `-error-limit=no|yes`：如果太多错误，则停止显示新错误？[yes]
4. `-error-exitcode=<number>`：如果发现错误则返回错误代码 [0=disable]
5. `-db-attach=no|yes`：当出现错误，valgrind 会自动启动调试器 gdb。[no]
6. `-db-command=<command>`：启动调试器的命令行选项 [gdb -nw % f % p]

### 适用于 Memcheck 工具的相关选项

1. `-leak-check=no|summary|full`：要求对 leak 给出详细信息？[summary]
2. `-leak-resolution=low|med|high`：how much bt merging in leak check [low]
3. `-show-reachable=no|yes`：show reachable blocks in leak check？[no]

## Valgrind 使用举例1

下面是一段有问题的 C 程序代码 test.c

```shell
#include <stdlib.h>
void f(void)
{
    int* x = malloc(10 * sizeof(int));
    x[10] = 0; //问题1：数组下标越界
} //问题2：内存没有释放

int main(void)
{
    f();
    return 0;
}
```

1. 编译程序 test.c

```shell
gcc -Wall test.c -g -o test
```

1. 使用 Valgrind 检查程序 BUG

```shell
valgrind --tool=memcheck --leak-check=full ./test
```

1. 分析输出的调试信息

```shell
==3908== Memcheck, a memory error detector.
==3908== Copyright (C) 2002-2007, and GNU GPL'd, by Julian Seward et al.
==3908== Using LibVEX rev 1732, a library for dynamic binary translation.
==3908== Copyright (C) 2004-2007, and GNU GPL'd, by OpenWorks LLP.
==3908== Using valgrind-3.2.3, a dynamic binary instrumentation framework.
==3908== Copyright (C) 2000-2007, and GNU GPL'd, by Julian Seward et al.
==3908== For more details, rerun with: -v
==3908==
--3908-- DWARF2 CFI reader: unhandled CFI instruction 0:50
--3908-- DWARF2 CFI reader: unhandled CFI instruction 0:50

/*数组越界错误*/
==3908== Invalid write of size 4
==3908== at 0x8048384: f (test.c:6)
==3908== by 0x80483AC: main (test.c:11)
==3908== Address 0x400C050 is 0 bytes after a block of size 40 alloc'd
==3908== at 0x40046F2: malloc (vg_replace_malloc.c:149)
==3908== by 0x8048377: f (test.c:5)
==3908== by 0x80483AC: main (test.c:11)
==3908==
==3908== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 14 from 1)
==3908== malloc/free: in use at exit: 40 bytes in 1 blocks.
==3908== malloc/free: 1 allocs, 0 frees, 40 bytes allocated.
==3908== For counts of detected errors, rerun with: -v
==3908== searching for pointers to 1 not-freed blocks.
==3908== checked 59,124 bytes.
==3908==

/*有内存空间没有释放*/
==3908== 40 bytes in 1 blocks are definitely lost in loss record 1 of 1
==3908== at 0x40046F2: malloc (vg_replace_malloc.c:149)
==3908== by 0x8048377: f (test.c:5)
==3908== by 0x80483AC: main (test.c:11)
==3908==
==3908== LEAK SUMMARY:
==3908== definitely lost: 40 bytes in 1 blocks.
==3908== possibly lost: 0 bytes in 0 blocks.
==3908== still reachable: 0 bytes in 0 blocks.
==3908== suppressed: 0 bytes in 0 blocks.
```

## Valgrind 使用举例2



```c++
#include <iostream>
#include <vector>
#include <memory>
using namespace std;

int main()
{
    // 1.new，but not free
    {
        auto *p = new int;
        *p = 2;
    }
    // 2.repeat delete
    {
        auto *p = new int;
        delete p;
        delete p;
    }
    // 3.new and free not match
    {
        auto *p = new int;
        free(p);
    }
    return 0;
}
```



```shell
# 无效的释放memory
==3238== Invalid free() / delete / delete[] / realloc()
# 在0x4da40d0是已经被释放的4个字节的块
==3238==    at 0x4845BD2: operator delete(void*, unsigned long) (vg_replace_malloc.c:1181)
==3238==    by 0x40119F: main (test.cpp:17)
# 第16行调用的delete释放了15行的new
==3238==  Address 0x4da40d0 is 0 bytes inside a block of size 4 free'd
==3238==    at 0x4845BD2: operator delete(void*, unsigned long) (vg_replace_malloc.c:1181)
==3238==    by 0x401189: main (test.cpp:16)
# 在第15行被分配
==3238==  Block was alloc'd at
==3238==    at 0x4842004: operator new(unsigned long) (vg_replace_malloc.c:487)
==3238==    by 0x40116F: main (test.cpp:15)
==3238== 
# 错误的free与new的匹配
==3238== Mismatched free() / delete / delete []
==3238==    at 0x4844B7B: free (vg_replace_malloc.c:989)
==3238==    by 0x4011B9: main (test.cpp:22)
==3238==  Address 0x4da4120 is 0 bytes inside a block of size 4 alloc'd
==3238==    at 0x4842004: operator new(unsigned long) (vg_replace_malloc.c:487)
==3238==    by 0x4011A9: main (test.cpp:21)
==3238== 
==3238== 
==3238== HEAP SUMMARY:
==3238==     in use at exit: 4 bytes in 1 blocks
==3238==   total heap usage: 4 allocs, 4 frees, 73,740 bytes allocated
==3238== 
# 看第一个上下文
# 4个字节没有释放，在test.cpp main函数中的第10行使用了new
==3238== 4 bytes in 1 blocks are definitely lost in loss record 1 of 1  #1 of 1：第一个上下文
==3238==    at 0x4842004: operator new(unsigned long) (vg_replace_malloc.c:487)
==3238==    by 0x401157: main (test.cpp:10)
==3238==  
==3238== LEAK SUMMARY:
==3238==    definitely lost: 4 bytes in 1 blocks
==3238==    indirectly lost: 0 bytes in 0 blocks
==3238==      possibly lost: 0 bytes in 0 blocks
==3238==    still reachable: 0 bytes in 0 blocks
==3238==         suppressed: 0 bytes in 0 blocks
==3238== 
==3238== For lists of detected and suppressed errors, rerun with: -s
# 先看所有总结：有三个错误在三个上下文
==3238== ERROR SUMMARY: 3 errors from 3 contexts (suppressed: 0 from 0)
```

