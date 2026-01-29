---
title: Dqueue
published: 2026-01-30T01:24:47+08:00
summary: "双端队列笔记"
cover:
  image: https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202601300127015.png
tags: [双端队列]
categories: '数据结构'
draft: false
lang: ''
---



# 代码

```c++
#pragma once
#include <iostream>
using namespace std;

# define dataBlockSize 10
# define ptrBlockSize 10

template<class DataType>
class Dqueue;

template<class DataType>
ostream& operator<<(ostream& os,const Dqueue<DataType>& temp);

template<class DataType>
class Dqueue
{
    public:
    //默认构造函数
    Dqueue()
    {
        this->blocks=new DataType*[ptrBlockSize]{};
        int ptrMiddle=ptrBlockSize/2;
        int dataMiddle=dataBlockSize/2;
        this->headDataPtr=dataMiddle;
        this->tailDataPtr=dataMiddle-1;
        this->headBlockPtr=this->tailBlockPtr=ptrMiddle;
        blocks[ptrMiddle]=new DataType[dataBlockSize];
        this->sum=0;
        this->blockSize=ptrBlockSize;
    }
    //拷贝构造函数
    Dqueue(const Dqueue<DataType>& others);
    //析构函数
    ~Dqueue()
    {
        if (this->blocks) {
            for (int i = this->headBlockPtr; i <= this->tailBlockPtr; i++) {
                if (this->blocks[i]) {
                    delete[] this->blocks[i];
                }
            }
            delete[] this->blocks;
        }
    }
    //添加元素
    void addHead(const DataType& data);
    void addTail(const DataType& data);
    //删除元素
    DataType deleteHead();
    DataType deleteTail();
    //判空函数,只读函数使用const
    bool empty() const;
    //打印函数配合重载<<
    ostream& printDqueue(ostream& os) const;
    friend ostream& operator<< <DataType>(ostream& os,const Dqueue<DataType>& temp);
    //clear函数
    void clear();
    //重载[]运算符,实现随机访问
    DataType& operator[](int index);
    //元素数目
    int size() const;

    private:
    int headDataPtr;
    int tailDataPtr;
    int headBlockPtr;
    int tailBlockPtr;
    DataType** blocks=nullptr;
    int sum=0;
    int blockSize=0;

    //扩容函数,对指针块扩大一倍
    void expandVolume();
};

template<class DataType>
ostream& Dqueue<DataType>::printDqueue(ostream& os) const
{
    if(this->sum==0) return os;
    if (this->headBlockPtr == this->tailBlockPtr)
    {
        for (int i = headDataPtr; i <= tailDataPtr; i++)
        {
            os << blocks[headBlockPtr][i] << "  ";
        }
        os << endl;
        return os;
    }
    //打印第一个数据块
    for (int i=headDataPtr;i<dataBlockSize;i++)
    {
        os<<blocks[headBlockPtr][i]<<"  ";
    }
    for (int i=this->headBlockPtr+1;i<tailBlockPtr;i++)
    {
        for (int j=0;j<dataBlockSize;j++)
        {
            os<<blocks[i][j]<<" ";
        }
    }
    //打印最后一个数据块
    for (int i=0;i<=tailDataPtr;i++)
    {
        os<<blocks[tailBlockPtr][i]<<"  ";
    }
    return os;
}

template<class DataType>
ostream& operator<<(ostream& os,const Dqueue<DataType>& temp)
{
    return temp.printDqueue(os);
}


template<class DataType>
Dqueue<DataType>::Dqueue(const Dqueue<DataType>& others)
{
    this->sum=others.sum;
    this->blockSize=others.blockSize;
    this->headDataPtr=others.headDataPtr;
    this->tailDataPtr=others.tailDataPtr;
    this->headBlockPtr=others.headBlockPtr;
    this->tailBlockPtr=others.tailBlockPtr;
    this->blocks=new DataType*[this->blockSize]{};
    for (int i=others.headBlockPtr;i<=others.tailBlockPtr;i++)
    {
        this->blocks[i]=new DataType[dataBlockSize];
        DataType* currArry=this->blocks[i];
        for (int j=0;j<dataBlockSize;j++)
        {
            currArry[j]=others.blocks[i][j];
        }
    }
}

template<class DataType>
void Dqueue<DataType>::clear()
{
    if (this->blocks) {
        for (int i = this->headBlockPtr; i <= this->tailBlockPtr; i++)
        {
            if (this->blocks[i]) {
                delete[] this->blocks[i];
                this->blocks[i] = nullptr;
            }
        }
        delete[] this->blocks;
    }
    this->blockSize = ptrBlockSize;
    this->blocks = new DataType*[this->blockSize]{};
    this->headBlockPtr = this->tailBlockPtr = this->blockSize / 2;
    this->headDataPtr = dataBlockSize / 2;
    this->tailDataPtr = dataBlockSize / 2 - 1;
    this->sum = 0;
    this->blocks[this->headBlockPtr] = new DataType[dataBlockSize];
}

template<class DataType>
void Dqueue<DataType>::expandVolume()
{
    int newBlockSize = this->blockSize * 2;
    DataType** newBlocks = new DataType*[newBlockSize]{};
    
    int oldBlockNum = this->tailBlockPtr - this->headBlockPtr + 1;
    int newHeadBlockPtr = (newBlockSize - oldBlockNum) / 2;
    
    for (int i = 0; i < oldBlockNum; i++)
    {
        newBlocks[newHeadBlockPtr + i] = this->blocks[this->headBlockPtr + i];
    }
    
    delete[] this->blocks;
    this->blocks = newBlocks;
    this->headBlockPtr = newHeadBlockPtr;
    this->tailBlockPtr = newHeadBlockPtr + oldBlockNum - 1;
    this->blockSize = newBlockSize;
}


template<class DataType>
void Dqueue<DataType>::addHead(const DataType& data)
{
    //第一种情况：队头数据块没有满
    if(this->headDataPtr!=0)
    {
        this->blocks[this->headBlockPtr][--this->headDataPtr]=data;
        this->sum++;
    }
    else
    {
        //第二种情况：队头数据块已经满了，需要对指针块扩容
        //判断指针块是否满
        if(this->headBlockPtr==0)
        {
            //指针块满了，直接扩容
            expandVolume();
        }
        
        //没有满或者扩容后，直接new一个新的数据块
        this->blocks[--this->headBlockPtr]=new DataType[dataBlockSize];
        this->headDataPtr=dataBlockSize-1;
        this->blocks[this->headBlockPtr][this->headDataPtr]=data;
        this->sum++;
    }
}

template<class DataType>
void Dqueue<DataType>::addTail(const DataType& data)
{
    //第一种情况：队尾数据块没有满
    if(this->tailDataPtr!=dataBlockSize-1)
    {
        this->blocks[this->tailBlockPtr][++this->tailDataPtr]=data;
        this->sum++;
    }
    else
    {
        //第二种情况：队尾数据块已经满了，需要对指针块扩容
        if(this->tailBlockPtr==this->blockSize-1)
        {
            //指针块满了，直接扩容
            expandVolume();
        }
        
        //没有满或者扩容后，直接new一个新的数据块
        this->blocks[++this->tailBlockPtr]=new DataType[dataBlockSize];
        this->tailDataPtr=0;
        this->blocks[this->tailBlockPtr][this->tailDataPtr]=data;
        this->sum++;
    }
}

template<class DataType>
DataType Dqueue<DataType>::deleteHead()
{
    if(this->sum==0)
    {
        throw "队列为空";
    }
    
    DataType temp = this->blocks[this->headBlockPtr][this->headDataPtr];
    
    // 情况1：队列中只有一个元素
    if (this->headBlockPtr == this->tailBlockPtr && this->headDataPtr == this->tailDataPtr)
    {
        delete[] this->blocks[this->headBlockPtr];
        this->blocks[this->headBlockPtr] = new DataType[dataBlockSize];
        this->headDataPtr = dataBlockSize / 2;
        this->tailDataPtr = dataBlockSize / 2 - 1;
    }
    // 情况2：当前数据块还有其他元素
    else if (this->headDataPtr < dataBlockSize - 1)
    {
        this->headDataPtr++;
    }
    // 情况3：当前数据块已空，跳到下一个块
    else
    {
        delete[] this->blocks[this->headBlockPtr];
        this->blocks[this->headBlockPtr] = nullptr;
        this->headBlockPtr++;
        this->headDataPtr = 0;
    }
    
    this->sum--;
    return temp;
}

template<class DataType>
DataType Dqueue<DataType>::deleteTail()
{
    if(this->sum==0)
    {
        throw "队列为空";
    }
    
    DataType temp = this->blocks[this->tailBlockPtr][this->tailDataPtr];
    
    // 情况1：队列中只有一个元素
    if (this->headBlockPtr == this->tailBlockPtr && this->headDataPtr == this->tailDataPtr)
    {
        delete[] this->blocks[this->tailBlockPtr];
        this->blocks[this->tailBlockPtr] = new DataType[dataBlockSize];
        this->headDataPtr = dataBlockSize / 2;
        this->tailDataPtr = dataBlockSize / 2 - 1;
    }
    // 情况2：当前数据块还有其他元素
    else if (this->tailDataPtr > 0)
    {
        this->tailDataPtr--;
    }
    // 情况3：当前数据块已空，跳到上一个块
    else
    {
        delete[] this->blocks[this->tailBlockPtr];
        this->blocks[this->tailBlockPtr] = nullptr;
        this->tailBlockPtr--;
        this->tailDataPtr = dataBlockSize - 1;
    }
    
    this->sum--;
    return temp;
}

template<class DataType>
bool Dqueue<DataType>::empty() const
{
    return this->sum==0;
}

template<class DataType>
int Dqueue<DataType>::size() const
{
    return this->sum;
}

template<class DataType>
DataType& Dqueue<DataType>::operator[](int index)
{
    if(index<0||index>=this->sum)
    {
        throw "索引错误";
    }
    //先找到指针块
    //第一块有多少元素
    int firstBlockNum=dataBlockSize-headDataPtr;

    //先看第一块有没有
    if(firstBlockNum>=index+1)
    {
        int trueIndex=headDataPtr+index;
        return blocks[headBlockPtr][trueIndex];
    }
    else
    {
        //再看剩下的相当于几个整块
        int blockNum=(index+1-firstBlockNum)/dataBlockSize;
        //再看剩下的元素数
        int lastElemNum=(index+1-firstBlockNum)%dataBlockSize;
        if(lastElemNum>0)
        {
            return blocks[headBlockPtr+blockNum+1][lastElemNum-1];
        }
        else
        {
            return blocks[headBlockPtr+blockNum][dataBlockSize-1];
        }
    }
}


```





# 常见问题

1. delete之后一定要将其变为nullptr，避免野指针

2. ```c++
   DataType** newBlocks=new DataType*[this->blockSize](nullptr);//
   ```

3. new数组时使用{}初始化，而不是()

4. 内存管理修正 (delete vs delete[])

   - 原问题 ：所有数据块都是通过 new DataType[dataBlockSize] 分配的，但原代码中使用了 delete 释放，这会导致内存泄漏或程序崩溃。
   - 修复 ：统一将数据块的释放方式修改为 delete[] 。

   即**一定要区分使用delete，与delete[]**

   


扩容逻辑优化 ( expandVolume )

- 原问题 ：原扩容逻辑在搬运数据时非常低效，且索引计算容易出错。
- 修复 ：重构了扩容函数。现在只需创建一个更大的块指针数组，并将现有的数据块指针平移到中间位置。这样只需拷贝指针，无需拷贝实际的数据元素，极大地提高了效率。
- 具体：原来的扩容只是简单的将所有元素都复制到新的数据块中，改进后，只需要换到更大的指针块，复用原数据块的数据，直接复制指向原数据块的指针不再次new



初始化与边界逻辑一致性

- 原问题 ： headDataPtr 和 tailDataPtr 的初始值设置不当，导致插入第一个元素时状态不统一，且 deleteHead / deleteTail 在删除最后一个元素时没有正确递减 sum 。
- 修复 ：将 tailDataPtr 初始值设为 dataMiddle - 1 ，并重写了 deleteHead 和 deleteTail ，确保它们在任何情况下都能正确更新 sum 和维护数据块的生命周期。
- 具体：就是原来的构造函数headBlockPtr以及TailBlockPtr都指向同一个块，headDataPtr，tailDataPtr都指向都一个数据地址



防止双重释放 (Double Free)

- 原问题 ：析构函数调用 clear() ，而 clear() 会重新分配内存，导致在对象销毁时产生不必要的分配或重复释放。
- 修复 ：重构了析构函数和 clear() ，确保析构时彻底释放所有资源。



**在使用delete前，一定要先用if判断是否存在**



代码复用逻辑差，在addHead中，如果头块只有一个元素我原来是

```c++
if(headBlockPtr==0)
{
    //扩容函数
    //然后将else逻辑复制
}
else
{
    //直接将headBlockPtr--
}

//改进后
if(``)
{
    扩容函数
}
统一执行else逻辑

```



# 关键代码

```c++
构造函数
析构函数
clear
扩容函数
add函数
```
