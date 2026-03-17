---
title: Linuxprobe
published: 2026-03-17T21:26:54+08:00
summary: "学习Linuxprobe过程笔记"
cover:
  image: https://i-blog.csdnimg.cn/blog_migrate/8e396597e76aeb10b2a7ac0940c8a7ed.jpeg#pic_center
tags: [linux]
categories: 'linux'
draft: false
lang: ''
---



# 学习资源

[Linux命令大全(手册) – 真正好用的Linux命令在线查询网站](https://www.linuxcool.com/)



# 一. 新手必备常用命令

## 1.命令格式

```shell
命令名称 [可选参数] [命令对象]
```



可选参数分为短格式和长格式，前者可写在一块并且参数名要缩写，后者不可写在一块

```shell
tar -zxvf
tar --help
```



## 2.常用组合键

1. tab:可以按一次以及两次
2. ctrl+c:强行终止一个命令
3. ctrl+D:表示键盘输入结束
4. ctrl+L:相当于clear



## 3.系统工作命令

1. echo

   在终端输出字符串或者变量提取后的值

   ```shell
   echo $SHELL
   /bin/bash
   ```

2. date

   显示系统时间以及日期，用于只查看时间以及更新时间

   ```shell
   date "+%Y-%M-%D %H:%M:%S"
   2026-03-17 21:39:32
   ```

3. timedatectl

   用于产看当前时间一些更基础东西：时区，系统时间，NTP服务等

4. reboot以及poweroff

5. wget（world wild web get）

   就是可以从终端下载网络文件（后面再讲）

6. ps

   查看进程状态    -aux

   补充状态信息：

   R：run正在运行的进程

   S：sleeping，在等待执行，有信号是可脱离该状态

   D：disk sleeping，无论有没有信号都不能脱离该状态

   Z：僵尸进程

7. pstree：有层次

8. top：区别于ps，他是动态的

9. nice：调整进程优先级（-20~19）低好

10. pidof：得到服务的进程号，一个服务可能有多个进程

11. kill/killall：杀死一个进程或一个服务



## 4.系统状态检测命令

概括：网卡/网络，系统内核，系统负载，内存使用情况，当前终端启用数量，历史登录记录，命令执行记录，救援诊断

1. ifconfig

   获取网卡配置以及网络信息（具体后面讲解）

2. uname：就是unix name，使用系统名称以及内核版本

3. uptime：查看系统负载信息

4. free：系统内存使用量

5. who：查看登录主机的用户终端信息

6. last：主机历史登录记录

7. ping：用于判断远程设备时候在线，或者两者之间的网络状态

8. tracepath：可用于检测两者之间经过的路由信息

9. netstat：显示网络连接，接口状态等信息

10. sos：生成关于系统架构信息用于诊断问题



## 5.查找定位文件命令

1. pwd：当前工作目录

2. cd：进入指定目录

   cd ..：返回上级目录

   cd ~:返回家目录

3. ls：显示目录中的文件信息   常用参数 -al

4. tree：结构化的展示目录结构

5. find：用于指定条件下查找文件以及目录

   ```shell
   find [查找范围] 寻找条件
   ```

6. locate:更块，但是要提前生成索引库文件

   ```shell
   updatedb //生成索引库
   locate whereis
   ```

7. whereis：用于查找命令二进制程序以及对应帮助文档位置

8. which：只查询命令程序位置



## 6.文本文件编辑命令

1. cat：用于查看纯文本文件（内容较少）通常加参数-n显示行号
2. more：用于查看纯文本文件（内容较多）
3. head/tail 用于只想要查看前几行，后几行内容
4. tr：translate，替换文本中指定的字符
5. wc：统计文本文件的行数，字节数，字数  对应参数是-l,-c,-w
6. stat：statue，查看文件的详细信息，包括访问时间，修改内容时间，修改文件属性时间
7. grep：是用途最广泛的文本搜索匹配工具，根据指定格式搜索和提取对应内容，常用参数-n(显示行号)，-v(反选信息)
8. cut：按照列提取内容
9. diff：文件是否不同
10. uniq：去除文本连续重复行
11. sort：指定规则排序



## 7.文件目录管理命令

1. touch：创建空白文件，修改文件属性

2. mkdir：创建目录，可以连续创建使用参数-p

3. cp：复制文件或者目录

4. mv：用于剪切或者重命名文件

5. rm：删除文件，-f强制删除，-r递归删除

6. file：查看文件类型，因为linux中没有后缀名

7. tar：打包指定的文件或者目录，注意-f后面一定要跟目标文件名

   ```shell
   tar -czvf target.tar.gz /root/test   //将root/test文件目录结构打包
   tar -xzvf target.tar.gz -c /root     //将root/test文件目录解压后放到root目录下
   ```

   























