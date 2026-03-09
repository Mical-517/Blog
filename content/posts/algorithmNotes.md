---
title: AlgorithmNotes
published: 2026-03-09T18:44:30+08:00
summary: "记录我的算法学习笔记与收获"
cover:
  image: https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202603091845907.png
tags: [algorithm]
categories: 'algorithm'
draft: false
lang: ''
---



# 开篇

## 如何估算时间复杂度

就是将一个算法每一个大步分解为时间复杂度为O(1)的每一个小步，然后估算每个小步的个数就可以了

## 常见的时间复杂度对应数据的范围

![image-20251109132427670](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202511091324743.png)

这里首先要明确为什么会超时，在算法比赛的测试机上，一般来说运行上限是10的8次方，所以如果从题目给出的数据范围来带入我们的算法后，分析出来的次数大于10的8次方，你们这个算法可能就不对

比如，数据范围是n<100000,为10的5次方，此时我们采用的算法应该是n*logn，不是n的平方，因为10的5次方在平方就是10的10次方，大于10的8次方

# 一.有关位运算

## 1.位运算符种类

1. &：全为1才是1
2. |：有1就为1
3. ^:相同为0，不同为1，性质：`a ^ a = 0`，`a ^ 0 = a`，且满足交换律和结合律。
4. ~：全部位取反
5. <<:左移运算符，用0补位
6. 右移运算符，>>,使用符号位补位，>>>使用0补位

## 2.关于常见位运算符的使用

```c++
//判断奇偶性，代替取模运算
1.n&1==1，为奇数，==0为偶数

//计算两个数的平均值，避免溢出
int average=(a^b)>>1+(a&b);
```



# 二.常见二分法以及二分法的变式

## 知识：二分法的模版写法

模版思路就是：

1. 如果一组数据，有一个性质或者说方法，可以将这个组内的元素完整分为两大类或者说范围
2. 我们要找的目标数据在这两个范围某个范围的边界上

如果满足这两个性质，就可以尝试使用二分法

注意事项：

**关于mid的计算**：mid=（left+right+1)/2；还是（left+right)/2,取决于**在你定义一个性质将数据分为两个范围的时候，看你的targetData在哪个范围的边界上**，如果在左边界限上，那么就是+1的

代码体现：

```c++
//如果target在做左边界上
while(left<right)
{
    //此时看left=mid，所以
    mid=(left+right+1)/2;
    if(arry[mid]<=target)
    {
		left=mid;
    }
    else
    {
        right=mid-1;
    }
}

//如果target在做左边界上
while(left<right)
{
    //此时看right=mid，所以
    mid=(left+right)/2;
    if(arry[mid]>=target)
    {
		rigth=mid;
    }
    else
    {
        left=mid+1;
    }
}
```

理解：

原理解释：（整数二分，因为数据是离散的，所以（right+left）/2,就要判断是采取上取整还是下取整

![image-20251110231500330](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202511102315440.png)

第一种情况：如果target是红线右端点

这样就把区间分为两段，然后假如说mid位于红线处，**注意这里mid比较的是我们定义的性质**，就表明target可能>=mid,因为target也是红线，所以是可能相等的，所以此时更新区间：left=mid；

如果mid在绿线处，mid一定是严格大于target的，所以right=mid-1;

**但是注意此时就有一个陷阱：**

如果我们的left=mid，假设现在只有两个元素，那么就有R=L+1；按照mid=（right+left）/2=L，此时如果满足left=right，那么执行L=L，就说区间并没有更新，原来的R仍然是R，L仍是L，**会死循环**，所以我们此时处理

mid=（left+right+1)/2;

同理我们可以分析第二种情况：

······

注：在做题的时候，我们只需要关注left=mid，还是right=mid，来确定mid是+1还是不+1，不用深入分析



## 题目练习

### 1.寻找局部最值问题

















