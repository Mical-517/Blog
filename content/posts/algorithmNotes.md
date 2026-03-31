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



## 二分答案的模版写法与使用

模版概述：

1. 就是在一个问题中，题目已给出一个变量y，然后要求一组范围内x的最值问题，**重要的是你能够看出来y与x之间有个函数的对应关系曲线**

![image-20260310124721027](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202603101247227.png)

比如上面，y就是与x有关的变量，这里目标值就是y=c，然后有一个**check函数用来输入利用y与x的关系来得到对应的y值，然后根据y是否在有效范围内判断是否在对应的解x是否早有效范围内，没有就使用二分缩小搜寻范围**

2. **还有一种就是y与x的关系是是否型的**，就是y的意义不是一个有效范围，而是就两种状态：是或者不是看第四题

## 题目练习

### 1.木材加工



[P2440 木材加工 - 洛谷](https://www.luogu.com.cn/problem/P2440)

已知：

1. 元素类型，int
2. 数据范围：8次方，使用int足够
3. 每一根原木的长度，以及**至少要切成k段**，注意这里至少，题目虽然没有明说，但是题目提到要切k段，就是说**每段的有效长度x**，至少要保证按照x切能够切的段数要>=k

求解：

每段长度的最大值



边界：只取一段，最大就是原木中最长的，所以每段长度就是在0~max之间

思路：

是说**每段的有效长度x**，至少要保证按照x切能够切的段数要>=k，这样才能够保证可以有k段，首先确定x，就是每段长度，

y就是可以切的段数，这里y与x关系就是，x变大，y变小，所以曲线线下

看上面模版的第二个图，就是本体的模版

确定二分的有效范围:0~max

代码：

```c++
#include <iostream>

using namespace std;

int n,k;//表示原木数量，小段数量
long sum=0;//表示和

bool check(int data)
{
    if(k*data<=sum)
    {
        return true;
    }
    else
    {
        return false;
    }
}

int main()
{
    cin>>n>>k;
    int temp;
    int treeMax=0;
    cin>>treeMax;
    sum+=treeMax;
    /*
    while(cin>>temp)
    {
        if(temp<treeMin)
        {
            treeMin=temp;
        }
        sum+=temp;
    }
    */
    int left=0;
    int right=treeMax;
    while(left<right)
    {
        int mid=(right+left+1)/2;
        if(check(mid))
        {
            left=mid;
        }
        else
        {
            right=mid-1;
        }
    }
    cout<<left<<endl; 
}

```



### 2.跳石头

[P2678 [NOIP 2015 提高组\] 跳石头 - 洛谷](https://www.luogu.com.cn/problem/P2678#ide)



已知：

1. L，N，M，总距离，石头总数，可以石头的最大数
2. 距离是int类型，因为一开始设置Di就是整数
3. D[i]，表示第i个石头到起点距离



求解：

如果移走的石头最大数量是M，想要是相邻两个石头之间的最小距离最大化，请输出这个最小距离的最大化值



边界范围：D[i]，只是从第i个石头到起点距离，但是从最后一个石头跳到终点也有一段距离，如果抽走所有石头，则最短距离最大化就是L，所以答案就在0~L之间



思路：

首先，要求一个最大值，并且 1.这个最短跳跃距离与所要已走的石头数有关系：距离大，移走的石头数就多   2.发现最多移走M块，所以移动块数y有个有效范围就是y<=M,这里就满足二分答案了，要求的和已知条件有关系，并且，一直条件有有效范围，所以x就一定也有有效范围，接下来就是在x有效范围选取一个最大值，找到一个函数来定义y与x的关系，发现，当我们确定x后，就可以对所有石头从头遍历，如果从begin到end距离小于x，就移动当前end所处的石头，end++

```c++
#include <iostream>
using namespace std;

int L,N,M;//距离，岩石数目，最多移走数目
int D[50005];//还要考虑第一块与最后一块
bool check(int distance)
{
    int begin=0,end=begin+1;
    int num=0;
    while(end<=N+1)
    {
        if(D[end]-D[begin]<distance)
        {
            num++;
            end++;
        }
        else
        {
            begin=end;
            end++;
        }
    }
    if(num<=M) return true;
    else return false;
}


int main()
{
    cin>>L>>N>>M;
    for(int i=1;i<=N;i++)
    {
        cin>>D[i];
    }
    D[0]=0;
    D[N+1]=L;
    int left=0,right=L;
    while(left<right)
    {
        int mid=(left+right+1)>>1;
        if(check(mid))
        {
            left=mid;
        }
        else
        {
            right=mid-1;
        }
    }
    cout<<left<<endl;
}
```



### 3.聪明的质检员

[P1314 [NOIP 2011 提高组\] 聪明的质监员 - 洛谷](https://www.luogu.com.cn/problem/P1314)

已知：

1. 一定要确认类型，看到y是要加在一起，那么就让位longlong型，s也为longlogn
2. n，m，s意义
3. 检验值yi的意义

求：

s-y绝对值的最小值



理解题意：

有一组石头，把他们分为m个区间，注意区间可以重复，计算每个区间里面符合要求（w>=W)的数量之和然后乘以对应的价值之和，得到的检验值y想加然后与s相减，求出最小值



思路：

我们可以发现，检验标准w越小，表示岩石越容易符合要求，所以y就越大，这里，y就随着w增大而增大，这里y的有效范围不是固定的一边，因为真正的判断条件是在一个W，对应的y要小于s就可以，所以我们可以自己选择s的上边是有效，还是下边是有效

**关键**：关于yi如何算，简单理解就是要找到一个区间符合条件的石头数量，然后乘以他们的价值之和，如果常规方法，就是对于m个区间每个都要计算一遍得到yi，但是我们知道区间是可以重复的，所以这是有些在前面已经判定的石块，还要再次判定一次，这是不必要的浪费，**优化：**我们可以采用递推的方法，递推做的不是 “算最终结果”，而是 **“存下所有可能的中间结果”**，递推的核心是 **“用前一步的结果，推导后一步的结果”**。

这道题的递推核心是 **“前缀和递推”**，我们拆解一下它的思维逻辑：

#### 1. 问题拆解

我们需要快速得到：**任意区间 \**[l,r]\** 内满足 \**wj≥W\** 的矿石数量** 和 **对应价值和**。

如果不预处理，每次算区间都要遍历 l 到 r，太慢。

#### 2. 递推的核心思想：把 “区间问题” 转化为 “前缀差问题”

定义两个数组：

- sn[i]：前 i 个矿石中，重量 ≥W 的矿石**总数量**。
- sv[i]：前 i 个矿石中，重量 ≥W 的矿石**总价值**。

**递推公式（状态转移）**：



```c++
// 第i个矿石满足条件，就继承前i-1的结果并+1/+价值；否则直接继承前i-1的结果
if (w[i] >= W) {
    sn[i] = sn[i-1] + 1;
    sv[i] = sv[i-1] + v[i];
} else {
    sn[i] = sn[i-1];
    sv[i] = sv[i-1];
}
```

#### 3. 区间查询（O (1)）

有了 sn 和 sv，任意区间 [l,r] 的结果就是**前缀差**：

- 数量：sn[r]−sn[l−1]
- 价值和：sv[r]−sv[l−1]

#### 4. 递推的 “巧妙” 之处

你用递推做的不是 “算最终结果”，而是 **“存下所有可能的中间结果”**。

-----



什么时候该用递推？只要满足以下 **3 个核心条件**，优先选递推：

#### 1. 问题具有**明确的状态转移关系**

即：**第 \**i\** 步的结果 = 第 \**i−1\** 步的结果 + 一个可计算的增量**。

- 本题：`sn[i]`（状态）由 `sn[i-1]` 和第 i 个矿石是否满足条件决定。
- 经典场景：斐波那契数列、前缀和 / 后缀和、差分恢复、动态规划（DP）的状态转移。

#### 2. 需要**预处理 / 累积统计**

当题目要求快速回答 “区间查询” 类问题（如区间和、区间最大值、区间计数）时，递推是预处理的最优解。

- 例如：区间和查询用前缀和（递推）、区间最值用 ST 表（递推）、差分数组还原（递推）。

#### 3. 数据量较大，需规避暴力超时

递推的时间复杂度通常是线性的 O(n)，能轻松处理 105∼106 级别的数据；而暴力法往往是 O(n2)，大数据下必超时。

```c++
#include <iostream>
#include <vector>
#include <utility>
#include <cstdlib>
using namespace std;

using LL=long long;
using Pair=pair<int,int>;
//岩石数目，区间数目，比较值
int n,m;
LL s;
//最小值
LL ans=1e18;

vector<Pair> stones;
vector<Pair> ranges;

bool check(int w)
{
    //前缀和（满足条件）
    vector<LL> w_count(n+1,0);
    vector<LL> v_count(n+1,0);
    for(int i=1;i<=n;i++)
    {
        w_count[i]=w_count[i-1];
        v_count[i]=v_count[i-1];
        if(stones[i-1].first>=w)
        {
            w_count[i]++;
            v_count[i]+=stones[i-1].second;
        }
    }
    //计算y
    LL y=0;
    for(int i=0;i<m;i++)
    {
        int l=ranges[i].first;
        int r=ranges[i].second;
        y+=(w_count[r]-w_count[l-1])*(v_count[r]-v_count[l-1]);
    }
    ans=min(ans,llabs(y-s));
    return y<=s;
}


int main()
{
    cin>>n>>m>>s;
    stones.resize(n);
    for(int i=0;i<n;i++)
    {
        cin>>stones[i].first>>stones[i].second;
    }
    ranges.resize(m);
    for(int i=0;i<m;i++)
    {
        cin>>ranges[i].first>>ranges[i].second;
    }
    int l=0,r=1000000;
    while(l<r)
    {
        int mid=(l+r)>>1;
        if(check(mid)) r=mid;
        else l=mid+1;
    }
    cout<<ans<<endl;
    return 0;
}
```



### 4.借教室

[P1083 [NOIP 2012 提高组\] 借教室 - 洛谷](https://www.luogu.com.cn/problem/P1083)

已知：

1. **下标从1开始**
2. n，m意义，d,t,s
3. 整数，对于和的变量使用longlong

求解：

这些订单是否会不满足教室容量，然后输出错误的编号



思路：

1. 容易想到暴力枚举，我们可以依次遍历每个订单，然后遍历每个订单的天数，用已有的教室依次减去，如果<0就表示错误，但是显示这是O(nm)，超时的，

2. 发现当订单数增加时显然各个天需要的教室会增加，所以随着订单数增加所需要的订单数总有一次会超过限制，所以如果y表示教室是否够，所以这就是**状态二分**，因为y就只有两种状态，可以或者不可以。并且题目还说了订单是顺序执行的，这天然满足二分的性质，因为教室的数量一定递减的。

   现在定义一个函数：输入订单编号，判断是否满足教室容量，所有我们就需要一个数组表示从第一个任务到当前任务每天所需要的教室总量，显然这一段描述就是当前第 \**i\** 步的结果 = 第 \**i−1\** 步的结果 + 一个可计算的增量,所以可以使用递推，但是我们发现这里递推还是一个O(mn),**这里就引出了一个新的递推方法**：详细看视频：[A06 二分答案 最好的套路_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV128411M7GT/?spm_id_from=333.1007.top_right_bar_window_history.content.click&vd_source=86d06d10a03dcd75fe3a632194bc3c3c)35分钟



```c++
#include <iostream>
#include <vector>
using namespace std;

int n,m;
vector<int> d;
vector<int> s;
vector<int> t;
vector<long> days;
vector<int> room;
bool check(int number)
{
    days.clear();
    days.resize(n+2,0);
    //计算前缀和
    for(int i=1;i<=number;i++)
    {
        int l=s[i];
        int r=t[i];
        days[l]+=d[i];
        days[r+1]-=d[i];
    }
    //开始求和
    for(int i=1;i<=n;i++)
    {
        days[i]+=days[i-1];
    }
    
    for(int i=1;i<=n;i++)
    {
        if(days[i]>room[i]) return false;
    }
    return true;
}

int main()
{
    cin>>n>>m;
    room.resize(n+1,0);
    days.resize(n+2,0);
    for(int i=1;i<=n;i++)
    {
        cin>>room[i];
    }
    s.resize(m+1,0);
    d.resize(m+1,0);
    t.resize(m+1,0);
    for(int i=1;i<=m;i++)
    {
        cin>>d[i]>>s[i]>>t[i];
    }
    int left=0,right=m;
    while(left+1<right)
    {
        int mid=(left+right)>>1;
        if(check(mid))
        {
            left=mid;
        }
        else
        {
            right=mid;
        }
    }
    if(check(left+1))
    {
        cout<<0<<endl;
        
    }
    else
    {
        cout<<-1<<endl<<left+1;
    }
    return 0;
}
```



### 5.刺杀大使

[P1902 刺杀大使 - 洛谷](https://www.luogu.com.cn/problem/P1902)

已知：

1. n行m列，下标从一开始
2. 类型为int
3. 只要到达第n行任意一个0其他的0就都可以到达
4. 到达第n行路径上的最大值（不是和）是一个士兵受伤的最大值，部队受伤的最大值是士兵当中受伤的最大值（就是每条路线的最大值相互比较）

求解：部队受伤的最小值



思路：

这就是典型的最大值最小化问题，我们发现，对与部队受伤的值x，将y设置为可以到达第n行，这里就有一个关系了，就是说如果在部队受伤为x的情况下，如果可以到达第n行，就代表这条路线的最大值<=x，随着x的增大，容错也就增大。当x在端点的时候，y恰好表示可以通行，无疑就是问题的解



与借教室类型相同，y都是是否型

```c++
#include <iostream>
#include <vector>
#include <cstring>
using namespace std;
//方向
int dx[]{-1,0,1,0};
int dy[]{0,1,0,-1};
int n=1001,m=1001;
int maze[1001][1001];
bool visit[1001][1001];

bool dfs(int x,int y,int injury)
{
    if(x==n) return true;
    visit[x][y]=true;
    //四个方向探索
    for(int i=0;i<4;i++)
    {
        int a=x+dx[i];
        int b=y+dy[i];
        if(a>=1&&b>=1&&a<=n&&b<=m&&visit[a][b]==false&&maze[a][b]<=injury)
        {
            if(dfs(a,b,injury)) return true;
        }
    }
    return false;
}



int main()
{
    cin>>n>>m;
    for(int i=1;i<=n;i++)
    {
        for(int j=1;j<=m;j++)
        {
            cin>>maze[i][j];
            visit[i][j]=false;
        }
    }
    int left=0,right=1000;
    int mid=0;
    while(left<right)
    {
        memset(visit,false,sizeof(visit));
        mid=(left+right)>>1;
        if(dfs(1,1,mid)) right=mid;
        else left=mid+1;
    }
    cout<<left<<endl;
    
    return 0;
}
```



### 6.银行贷款

[P1163 银行贷款 - 洛谷](https://www.luogu.com.cn/problem/P1163)

已知：

1. 贷款金额，还款金额long，mounth是int
2. **注意他说的是每月固定偿还**，所以说算利息的时候，是按照换一部分之后的金额算的



求解：利率

思路：

题目中有一个不得不，这表示每月最低还款金额，所以说如果将还款金额设定为y，那么y就要>=wo,将利率设为x，那么随着x增加，最低还款金额就要上升，所以y随x增大而增大，所以这是一个利率最小值最大化问题，当利率足够小的时候，他在w0情况下当然可以还清，所以check函数，就是监测，在当前利率下，如果还完mounth之后，如果还剩，就表示利率taida

```c++
#include <iostream>
#include <iomanip>
using namespace std;

long amount,interestAmount;
int mounth;

bool check(double interest)
{
    double sum=amount;
    for(int i=0;i<mounth;i++)
    {
        sum=sum*(1+interest)-interestAmount;
    }
    if(sum<=0)
        return true;
    else return false;
}
int main()
{
    cin>>amount>>interestAmount>>mounth;
    double left=0,right=3.0;
    while(left<right)
    {
        double mid=(left+right+0.001)/2;
        if(check(mid)) left=mid;
        else right=mid-0.001;
    }
    
    cout<<fixed<<setprecision(1);
    left*=100;
    cout<<left<<endl;
    return 0;
}


```

## 总结

![image-20260313120211023](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202603131202213.png)



# 三. 前缀和与差分题目



## 1. 一维前缀和

概念理解：

![image-20260313141257174](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202603131412326.png)



就是如果题目对与一个区间想加很多次，但是没有要求我们去求过程中的值，只是要求我们计算最后的值，此时，这个方法比较有用。还有一种方法是如果在多个区间**加上固定的一个值，此时有一种特殊的前缀和的求法，类似二.4借教室**

## 2.二维前缀和

![image-20260313141922006](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202603131419114.png)



**关键：在关于前缀和的题目中最好下标从一开始，这样可省略判断i<=0情况**





## 题目

### 1.求区间和

[P8218 【深进1.例1】求区间和 - 洛谷](https://www.luogu.com.cn/problem/P8218)

就是常规模版用法

### 1.激光炸弹



[P2280 [HNOI2003\] 激光炸弹 - 洛谷](https://www.luogu.com.cn/problem/P2280)

关键：**虽然题目给出x，y是从0开始，但是我们建模的时候，让他们偏移一维，都从1开始简便后续逻辑判断**

```c++
#include <iostream>
using namespace std;
const int N=5005;

int n,m;
int v[N][N];



int main()
{
    cin>>n>>m;
    int x,y,value;
    for(int i=0;i<n;i++)
    {
        cin>>x>>y>>value;
        //可以让x，y偏移一个单位空出0,0，以便后续方便求前缀和
        x++;
        y++;
        v[x][y]+=value;//同一个位置有多个目标
    }
    //计算前缀和
    int sum[N][N];
    for(int i=1;i<=5001;i++)
    {
        for(int j=1;j<=5001;j++)
        {
            sum[i][j]=sum[i-1][j]+sum[i][j-1]-sum[i-1][j-1]+v[i][j];
        }
    }
    //遍历炸弹的位置,以右下角为准
    int max=0;
    for(int i=m;i<=5001;i++)
    {
        for(int j=m;j<=5001;j++)
        {
            int w=sum[i][j]-sum[i-m][j]-sum[i][j-m]+sum[i-m][j-m];
            max=(max>w)?max:w;
        }
    }
    cout<<max<<endl;
    return 0;
}
```



## 3.树上前缀和

![image-20260314175041874](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202603141750983.png)

思路理解：

如果要求一个树上前缀和，就要：

1. 实现一个树结构：移步到algorithmB篇（使用链式前向星构建）
2. **关键：找到这两个节点的最近公共祖先**：[D09 倍增算法 P3379【模板】最近公共祖先（LCA）——信息学奥赛算法_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1vg41197Xh/?spm_id_from=333.337.search-card.all.click&vd_source=86d06d10a03dcd75fe3a632194bc3c3c)

![image-20260314160049392](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202603141600561.png)

关键：

![image-20260314180105515](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202603141801600.png)

![image-20260314180147771](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202603141801876.png)





## 题目：

### 1.求和

[P4427 [BJOI2018\] 求和 - 洛谷](https://www.luogu.com.cn/problem/P4427)

思路：

首先构建一个树，发现这是一个多次询问，k次方前缀和的问题，所以，我们可以提前把每个从根节点到当前节点的深度的k次方之和求出来，因为k是不定的，所以干脆把所有情况都罗列出来，就有了s【】【】数组，然后找到最近的公共祖先，使用树上前缀和模版，就可得到x到y路径上所有节点的深度的k次方之和



问题：

1. 构建树
2. 如何得到每个节点的从根到当前深度的k次方之和
3. 如何得到最近最近公共祖先

```c++
#include <bits/stdc++.h>
using namespace std;

using LL=long long;
const LL mod=998244353;

//使用链向前向星存储无向图
const int N=3e5+5;
int n,m;
//为什么2*N，因为是无向图，每个边都要存两次
int h[N],to[2*N],ne[2*N],tot;//节点-边编号，边编号-终点，边编号-相同起点边编号，边编号

int fa[N][22];//fa[x][i]表示x节点向上跳2^i层的祖先节点
int dep[N];//dep[x]表示x节点深度
//以上是倍增法求公共祖先的模版，下面两个是针对此题的改进
LL mi[55];//mi[j]表示儿子节点v此时的dep[v]的j次幂（相当于是一个临时变量）
LL s[N][55];//s[x][j]表示根节点到x节点之间所有节点深度的j次幂之和


void add(int a,int b)
{
    to[++tot]=b;
    ne[tot]=h[a];
    h[a]=tot;
}

//当前节点x，父节点f，用来完成fa表，以及计算到当前节点的路径j次幂之和
//dfs在可以遍历到每个节点，所以可以针对每个节点完善fa表，以及计算前缀和
void dfs(int x,int f)
{
    for(int i=1;i<=20;i++)
    {
        fa[x][i]=fa[fa[x][i-1]][i-1];
    }
    //遍历x每一个儿子节点
    for(int i=h[x];i;i=ne[i])
    {
        int y=to[i];
        //不走回头路，只向下去走
        if(y!=f)
        {
            fa[y][0]=x;
            dep[y]=dep[x]+1;
            for(int j=1;j<=50;j++)
            {
                mi[j]=mi[j-1]*dep[y]%mod;
            }
            for(int j=1;j<=50;j++)
            {
                s[y][j]=(mi[j]+s[x][j])%mod;
            }
            dfs(y,x);
        }
    }
}

//求最近公共祖先
int lca(int x,int y)
{
    if(dep[x]<dep[y]) swap(x,y);
    //现将x，y变为同一层
    for(int i=20;i>=0;i--)
    {
        if(dep[fa[x][i]]>=dep[y]) x=fa[x][i];
        
    }
    if(x==y) return y;
    for(int i=20;i>=0;i--)
    {
        if(fa[x][i]!=fa[y][i])
        {
            x=fa[x][i];
            y=fa[y][i];
        }
    }
    return fa[x][0];
}



int main()
{
    cin>>n;
    int a,b;
    for(int i=1;i<n;i++)
    {
        cin>>a>>b;
        add(a,b);
        add(b,a);
    }
    mi[0]=1;
    dfs(1,0);//倍增预处理，dep，fa，mi，s数组
    cin>>m;
    for(int i=1,x,y,k;i<=m;i++)
    {
        cin>>x>>y>>k;
        int z=lca(x,y);
        LL ans=(s[x][k]+s[y][k]-s[z][k]-s[fa[z][0]][k]+2*mod)%mod;
        cout<<ans<<endl;
    }
    return 0;
}
```

### 2.最近公共祖先

[P3379 【模板】最近公共祖先（LCA） - 洛谷](https://www.luogu.com.cn/problem/P3379)

关键如何使用数据结构存储图结构以及怎样来使用全局常量

```c++
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 5e5 + 10;
const int LOG = 20;

vector<int> adj[MAXN]; // 邻接表：adj[u] 存 u 的所有邻接节点
int fa[MAXN][LOG];
int dep[MAXN];
int N, M, S;

void dfs(int u, int father) {
    dep[u] = dep[father] + 1;
    fa[u][0] = father;
    // 倍增预处理：2^i 级祖先
    for (int i = 1; i < LOG; ++i) {
        fa[u][i] = fa[fa[u][i-1]][i-1];
    }
    // 遍历子节点
    for (int v : adj[u]) {
        if (v != father) {
            dfs(v, u);
        }
    }
}

int lca(int u, int v) {
    if (dep[u] < dep[v]) swap(u, v);
    // 1. 先把 u 跳到和 v 同一深度
    for (int i = LOG-1; i >= 0; --i) {
        if (dep[fa[u][i]] >= dep[v]) {
            u = fa[u][i];
        }
    }
    if (u == v) return u;
    // 2. 一起向上跳，直到 LCA 的下一层
    for (int i = LOG-1; i >= 0; --i) {
        if (fa[u][i] != fa[v][i]) {
            u = fa[u][i];
            v = fa[v][i];
        }
    }
    return fa[u][0];
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cin >> N >> M >> S;
    // 读入 N-1 条边
    for (int i = 0; i < N-1; ++i) {
        int x, y;
        cin >> x >> y;
        adj[x].push_back(y);
        adj[y].push_back(x);
    }
    // 预处理根节点 S
    dep[0] = 0;
    dfs(S, 0);
    // 处理查询
    for (int i = 0; i < M; ++i) {
        int x, y;
        cin >> x >> y;
        cout << lca(x, y) << '\n';
    }
    return 0;
}
```



## 4.差分

![image-20260316174038846](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202603161740103.png)

**注意：这里差分序列的容量大小应该为n+1，因为要确保最后一个a序列的最后一位差分也可包含在内**



## 题目：

### 1.一维差分

[P4552 [Poetize6\] IncDec Sequence - 洛谷](https://www.luogu.com.cn/problem/P4552)

思路关键:

题目提到目标是让序列全部相同，并且是多次操作只看结果，所以联想到**转换为差分就是让差分序列结果的b2到bn全部为0就可以了**，转换为两点操作，然后就是确定是哪两点了，因为不需要看具体操作的是哪一个序列区间，并且一次只能增1减1，所以可以直接使用正负之和，来让内部的相互抵消，最后还剩下的有选两边的点进行消除到0

![image-20260316174710431](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202603161747732.png)



### 2.二维差分

[P3397 地毯 - 洛谷](https://www.luogu.com.cn/problem/P3397)

思路：

先想暴力破解：就是枚举m个地毯，然后内部循环一个地毯区间让每一位都+1，但是为O(mn^2)；

这里看到内部循环是让每一个地毯区间都加上1，就是让n行序列区间加上1，**并且是多次访问这个序列**，所以可变为差分的形式

**注意这里坐标是逻辑还是物理序号**





# 四.01分数规划（包含背包问题）



## 1.首先理解01分数规划思想

首先理解01分数规划：

就是一个有一组数据，然后每个数据有a,b两种属性，从中抽取数据使得这组数据a之和/b之和，最大或者最小

**可以转换为**：给出a，b让求这组物品的是0，还是1（0表示不取，1表示取）的这样的序列，使得和之比最值问题





![image-20260317184134947](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202603171841226.png)

既然是一个最值问题，那么线稿使用二分法求取最大值，这里假设我们去中间答案值x，如果标准答案和之比>=x,就表示，这个答案下，取数据是可以达到这个比较小的和之比的，所以x有效，那么x可以继续增大，所以答案在x右边，**但是每个数据都有两个属性，我们都要求和，不方便**，所以将不等式化简，可以发现，最后的不等式，只要每个数据的新的权值之和<=0就表明x有效，所以**我们可以吧这个新权值代替原来的数据属性，只需要挑选满足条件的物品，然后计算它的新权值之和与0的关系就可以判断x是否有效**



**关键：**

1. 明确属性，为每一个属性定义一个新的权值
2. 公式这里要求权值之和，但是挑选哪些物品来求之和，这**就设计到物品选取的约束条件，下面选取01背包问题作为典型的数据选取约束例子**



## 2.关于选取规则（01背包问题）

如果涉及到数据的选取，并且选取的这组数据某一个属性要在一个区间范围之内，或者说某一个属性之和是固定的不能超过，这是就是一个背包问题，在细分根据物品是否可以被重复选取有01（选一次），重复背包（可以多次选择）两种变式



### 1. 01背包问题

首先是状态定义：

dp[j]:表示在容量为j下，对应另一种属性线性和的最大值

初始化dp:

1. 范围:范围就是题目要求的最大容量是多少

2. dp[0]=0：这是确定的，总容量为0，价值就是0

   但是从容量1开始到W，都要设置一个最小值，**来表示这个容量状态下，我还没有一种确定的方案，或者说我还没定义好这个容量的状态，自然就无法使用这个状态来更新别的状态**



模版理解：

```c++
//定义初始化dp
int dp[10000];
for(int i=1;i<=100000;i++) dp[i]=-1e9;

//两层循环
//第一层遍历每个物品：选择或者不选择
for(int i=1;i<=n;i++)
{
    //第二层循环遍历每一个容量，利用当前容量来作为状态起点，更新更大的容量的状态
    for(int j=W;j>=0;j--)
    {
        if(dp[j]<=-1e9) continue;
        int k=j+w[i];
        dp[k]=max(dp[k],dp[j]+物品i的新权值);
    }
}
```





**关于内层循环的详细阐述：**

1. 为什么要遍历内层循环：

   这是根据状态转移方程决定的：对于第i个物品，容量j+w[i]=k

   则: `dp[k]=max(dp[k],dp[j]+w[i])`,对与一开始的dp[k]对与物品i，它有两种选择，选就是要看对应原来的小容量状态下的权值之和加上当前物品的权值，不选就是延续之前的dp[k],**所以更高容量状态的dp首先要从低的状态然后比较的到是否需要选择**

2. 既然要循环从低状态作为起点去更新高状态，为什么倒序

   因为如果正序遍历，那么每一个物品可以被选择两次，这在01是不允许的

   解释：如果从低容量a状态遍历，那么更新一个高容量状态b后，由于正向遍历，所以会到b，以b状态可能又会更新状态c，但是状态b已经选取了外层的物品i，如果c在b的基础有一次选择（更新），那么物品i就被选择两次，**所以要确保更新的高状态不会再次变为状态起点，所以是倒序遍历**





## 题目：

### 1.洛谷4377

[P4377 [USACO18OPEN\] Talent Show G - 洛谷](https://www.luogu.com.cn/problem/P4377)



首先分析：

一组奶牛数据，每个数据有重量，才能两个属性

数据约束：

1. 这组数据的重量之和要>=W_COUNT
2. 要求出满足数据约束的物品的和之比最大的值



首先一组数据求取求相应属性和之比的最大值：01分数问题

那么  根据分析：如果使用二分定义为x，那么不等式经过化简就可以得到让才能vi-重量wi*m作为每一个数据的新权值，所以就是求在数据约束下的这一组物品的最大新权值之和是多少

分析数据约束：就是给定一个容量要求这组数据的w之和不能<这个容量，还要让另一个新权值之和最大，所以这也是个背包物品选取问题





# 五.双重指针

模版速览

[A18 双指针（尺取法）_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Mp4y1E74K/?spm_id_from=333.1387.collection.video_card.click&vd_source=86d06d10a03dcd75fe3a632194bc3c3c)

![image-20260320201149108](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202603202011333.png)

双重指针主要作用在对与一个序列而言，它的有效范围是就是要从区间（有点类似前缀和的感觉）里做计算，或者满足什么条件来进行得到的，通常来说，如果从题目给的信息来看的话，答案可能要从一个序列之中顺序查找，那么此时可以考虑双重指针来规避双重循环每一个都要遍历的形式，提高性能。



## 题目

### 1.连续正整数的和

[P1147 连续正整数和 - 洛谷](https://www.luogu.com.cn/problem/P1147)

思路：

首先给定一个整数，那么答案就要从1开始自然数这个序列查找的一个区间（因为是连续的），那么就可以考虑使用双指针，分别表示两端



### 2.逛画展

[P1638 逛画展 - 洛谷](https://www.luogu.com.cn/problem/P1638)

![image-20260320202019203](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202603202020379.png)

关键是变量的选取设置以及建模思想



### 3.AB数对

[P1102 A-B 数对 - 洛谷](https://www.luogu.com.cn/problem/P1102)

关键是思想：

要找出一对A，B满足A-B=C，就是可以枚举B，求取一个A满足B+C，所以可以枚举B，然后确定有效区间两端，然后更新b的时候，两端可以不变，因为我们让序列有序了，当b变大，所以对应的a一定较之前变大，所以a就可以一直往前遍历不用回头





