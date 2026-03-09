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

   **重新描述条件1**：就是如果有一个函数（或者说方法），当我们把这一组数据当中的某一个数据或者两个数据带入的时候，通过这个方法的返回值与题目条件的限定下，可以知道我们的目标答案在带入数据的左侧还是右侧，此时就是二分法

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

[162. 寻找峰值 - 力扣（LeetCode）](https://leetcode.cn/problems/find-peak-element/description/)

思路：

已知条件：

1. 元素类型：int
2. nums相邻元素不重复（重要）
3. 峰值定义：就是左右两边的数字大
4. 数组不有序

边界：就是两边只有一个相邻位置可以比较



求解：任意一个峰值的下标



思路：

先判断边界数字是不是峰值，如果不是那就意味着边界是这样的：a[0]到a[1]是向上增长趋势，a[n-2]到a[n-1]是向下增长趋势，从向上到向下，所以中间肯定有一个我们的目标值，所以**我们的目标值的区间是这样的**，两边一个向上，一个向下，所以我们定义一个方法，将数据带入就会得到他是向上还是向下，然后一步一步收缩



代码：

```c++
class Solution {
public:
    int findPeakElement(vector<int>& nums) {
        if(nums.size()==1)
        {
            return 0;
        }
        //先判断两边
        if(nums.at(0)>nums.at(1))
        {
            return 0;
        }
        if(nums.at(nums.size()-1)>nums.at(nums.size()-2))
        {
            return nums.size()-1;
        }
        //判断中间元素
        int left=1;
        int right=nums.size()-2;
        int mid=0;
        while(left<right)
        {
            mid=(left+right+1)/2;
            //这里这个条件<就是定义的方法，mid就是判断的数值
            if(nums.at(mid)<nums.at(mid-1))
            {
                right=mid-1;
            }
            else
            {
                left=mid;
            }
        }
        return left;
    }
};
```





### 2.一元三次方程问题

[P1024 [NOIP 2001 提高组\] 一元三次方程求解 - 洛谷](https://www.luogu.com.cn/problem/P1024)



已知条件：

1. 元素类型double
2. 数据范围 -100-100，实数范围
3. *f*(*x*)=0，若存在 2 个数 *x*1 和 *x*2，且 *x*1<*x*2，*f*(*x*1)×*f*(*x*2)<0，则在 (*x*1,*x*2) 之间一定有一个根。
4. 根与根之差的绝对值 ≥1代表，寻到根的范围大小就是1个单位，两个根之间的距离是>=1，所以为了保证搜寻范围只有一个根，就要保证搜寻范围是一个一个单位来搜寻的



求解：

满足方程的三个解，要从大到小pailie



搜寻边界：从i~i+1，如果边界i就是根，则i到i+1就没有必要搜寻了

思路：

因为**根与根之差的绝对值 ≥1，并且从大到小排列**，所以从i=-100开始每一次都是搜寻i~i+1范围内的根

搜寻策略：定义二分方法：*f*(*x*1)×*f*(*x*2)<0，则在 (*x*1,*x*2) 之间一定有一个根，所以f(x)想乘就是定义的函数用来找到目标在那一侧



```c++
#include <iostream>
using namespace std;

double a,b,c,d;
//带入x得到方程的结果
double result(double x)
{
    return a*x*x*x+b*x*x+c*x+d;
}

//已知一个范围有根，得到根
double get(double l,double r)
{
    //实数二分
    double mid=0;
    while(r-l>0.0001)
    {
        mid=(l+r)/2;
        if(result(mid)*result(l)<0)
        {
            r=mid;
        }
        else
        {
            l=mid;
        }
    }
    return l;
}

int main()
{
    cin>>a>>b>>c>>d;
    for(int i=-100;i<100;i++)
    {
        //先验证范围边界是不是根
        double y1=result(i),y2=result(i+1);
        if(!y1)
        {
            printf("%.2lf ",1.0*i);
        }
        if(y1*y2<0)
        {
            printf("%.2lf ",get(i,i+1));
        }
    }
    return 0;
}
```

知识点：关于限定小数位数，使用printf打印



















