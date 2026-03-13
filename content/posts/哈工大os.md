---
title: 哈工大os
published: 2026-03-13T20:43:56+08:00
summary: "观看笔记"
cover:
  image: https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202603132045971.png
tags: [os]
categories: 'os'
draft: false
lang: ''
---



# 1.现在电脑开机通电了

首先，os是一个软件，运行就要装在memory，如何载入的呢？计算机有一点固有的引导程序BIOS，就使用来引导os开始运行工作

![image-20260313210515294](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202603132105316.png)

## 第一阶段：开始运行bootsect.s

bootsect程序，（一开始cpu处于实时模式，只能寻址16位），首先将自身程序转移到另一个位置，空出当前位置（后面解释），然后将setup程序模块装载到内存，将system模块装载到0地址处，这就是为什么一开始bootsect就要腾出空间，防止覆盖原来的程序。

主要作用：加载setup程序模块。



## 第二阶段：运行setup模块（完成os启动前的设置）



setup模块初始化一些硬件参数比如内存大小，方便后续os初始化时分配内存合理，使用一条高级汇编指令（cpu变为保护模式，32位），然后转移指令开始执行system（位于0地址处）



## 第三阶段：执行system程序

system是有许多模块组合的，这时候，先执行什么就尤为重要。

先执行head.s文件，之后进入main函数进行os的初始化（初始化一切os可掌握的资源情况）**注意这里main函数永远不会退出，很好理解，我们使用pc一定是要断电之后，os才不工作**。







