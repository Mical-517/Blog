---
title: Gdb调试
published: 2026-04-18T15:02:37+08:00
summary: "关于gdb调试的相关学习与记录"
cover:
  image: https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202604181505676.png
tags: [gdb]
categories: '工具链'
draft: false
lang: ''
---



# 引言：

如何在实际的、复杂的C/C++项目（如Redis）中，高效、精准地使用GD

学习过程类比手速过程：

- 准备手术刀：调试前的“无菌环境

  确保被调试的程序包含完整的调试信息，并且处于最适合调试的编译状态（关闭优化）。这是所有后续调试工作的基础，否则GDB将无法提供准确的代码行、变量名和函数调用信息。

- 选择手术方案：三种接入方式

  掌握针对程序未启动、程序已运行、程序已崩溃这三种不同状态，如何正确地将GDB附加到目标上进行调试。这是应对实际开发中各种调试需求（如在线调试、死机分析）的关键。

- 掌握基础刀法：核心调试命令

  熟练使用一组最常用、最核心的GDB命令，实现对程序执行流程的精确控制（运行、暂停、继续）和对程序内部状态（代码、变量、调用关系）的实时观察。这是交互式调试的日常操作。

- 高级诊断与病灶管理

  学习对断点进行高效管理（查看、启用、禁用、删除），使用临时断点满足特殊需求，并了解如何利用像Visual Studio这样的IDE来辅助管理复杂的项目源码，提升调试前的代码分析效率。

  

# B1.调试环境的搭建与准备

![image-20260418164121709](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202604181641759.png)



## C1：调试符号-g

目的：为了能让GDB清晰地展示每一行代码、变量名、函数调用堆栈等信息，被调
试的程序必须包含调试符号。 调试符号是连接机器指令与源代码的“地图”。

```shell
# 编译单个源文件
gcc -g -o0 my_program my_program.c
# 编译C++程序
g++ -g -o0 my_program my_program.cpp
# 在Makefile或CMakeLists.txt中对应位置添加 -g 标志

-------------------------

gcc -g -o0 my_program.c -o my_program
```

-o0是为了关闭编译器优化，防止源码行号与编译器认为行号不同



# B2:启动调试的三种核心场景

![image-20260418164859351](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202604181648394.png)



## C4:直接启动调试：gdb filename

核心概念： gdb filenamefilename 是需要调试的可执行程序的文件名。此命令会启动GDB，并加载指定的程序，但程序并未立即运行，而是等待用户输入调试命令（如 run ）后才开始执行。

⚠️常见错误与注意事项

1. 找不到文件：确保当前目录或指定路径下存在可执行文件，且具有执
   行权限。
2. 缺少调试信息：如果编译时未使用 -g 选项，GDB会提示(no
   debugging symbols found)，此时无法进行源码级调试。
3. 程序参数传递：可以在run命令后直接传递程序运行所需的命令行参
   数。



## C5:附加到运行中进程：gdb attach pid

当程序已经在运行（例如一个长期运行的服务如Redis、Nginx，或一个卡住的进程），我们需要在不重启程序的情况下介入调试。



核心概念： gdb attach pidpid 是目标进程的进程ID（Process ID）。此命令会将GDB“附加”到正在运行的进程上，立即暂停该进程的执行，并将控制权交给GDB。
调试结束后，可以使用 detach 命令让程序继续独立运行，或 kill 命令终止它。

![image-20260418165352393](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202604181653454.png)

适用场景：线上服务问题诊断（高CPU、死锁）、分析已运行程序的状态、调试无
法轻易重启的进程。



## C6：生成与分析Core文件：ulimit 与 Core Dump

当程序已经崩溃（例如段错误Segmentation Fault），我们需要事后的“尸检报告”来分析原因。Core Dump（核心转储）文件就是程序崩溃瞬间的完整内存快照。

核心概念：Core DumpCore Dump是程序崩溃时操作系统生成的一个文件，包含了崩溃瞬间的进程内存映像、寄存器状态、堆栈信息等。它是事后调试（Post-mortem Debugging）的关键。默认情况下，许多系统禁止生成Core文件，需要手动开启。



启用core文件生成操作步骤：

1. 检查当前限制：

   ```shell
   ulimit -c
   0
   # 输出0表示禁止生成core文件
   ```

2. 解除限制（临时）

   ```shell
   ulimit -c unlimited
   # 设置后，允许生成任意大小core文件，进队当前shell会话成立
   ```

   永久生效：将 ulimit -c unlimited 添加到 ~/.bashrc 或/etc/security/limits.conf 文件中。对于系统服务，可能需要在systemdservice文件中配置LimitCORE=infinity。

3.  指定Core文件名称和路径

   ```shell
   $ sudo sysctl -w kernel.core_pattern=/tmp/core-%e-%p-%t kernel.core_pattern = /tmp/core-%e-%p-%t
   # 格式说明：%e可执行文件名，%p进程ID，%t崩溃时间戳。这可以防止Core文件被覆盖，并便于管理。
   ```



## C7:调试Core文件：gdb filename corename



在成功生成Core文件后，我们可以使用GDB加载它，像“时间旅行”一样回到程序崩溃的瞬间，检查当时的程序状态。



{{< pdf url="https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/pdf/gdb.pdf" >}}





























