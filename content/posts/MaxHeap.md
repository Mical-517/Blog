---
title: MaxHeap
published: 2026-02-04T10:27:46+08:00
summary: "用数组实现最大堆的思想以及方法"
cover:
  image: https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202601291712327.png
tags: [heap]
categories: '数据结构'
draft: false
lang: ''
---

# 代码

```c++
#pragma once
#include <iostream>
#define MAX_SIZE 100
using namespace std;

template<class DataType>
class MaxHeap;

//利用数组实现最大堆
template<class DataType>
class Data
{
    public:
    friend class MaxHeap<DataType>;
    Data(int p=0,DataType d=DataType()):priority(p),data(d){}
    // 访问器：允许外部读取 data 和 priority（只读）
    int getPriority() const { return priority; }
    DataType getData() const { return data; }

    // 修正：operator= 作为非静态成员函数
    Data& operator=(const Data& rhs) {
        if (this != &rhs) {
            priority = rhs.priority;
            data = rhs.data;
        }
        return *this;
    }

    private:
    int priority=0;
    DataType data;

    bool operator<(const Data& other) const
    {
        return this->priority<other.priority;
    }
};

template<class DataType>
class MaxHeap
{
    public:
    MaxHeap():size(0){}
    //插入元素
    void insert(DataType& data,int priority);
    //删除堆顶元素
    void remove();
    //获取堆顶元素
    Data<DataType> getMax() const;
    private:
    Data<DataType> heap[MAX_SIZE];
    int size=0;
    //插入元素辅助函数swap
    void swap(Data<DataType>& a,Data<DataType>& b);
};

template<class DataType>
void MaxHeap<DataType>::insert(DataType& data,int priority)
{
    if(this->size>=MAX_SIZE)
    {
        cout<<"Heap is full"<<endl;
        return;
    }
    // 插入新元素到数组的末尾
    this->heap[this->size]=Data<DataType>(priority,data);
    this->size++;

    //上浮新插入的元素
    int index=this->size-1;
    while(index!=0)
    {
        int parentIndex=(index-1)/2;
        if(this->heap[parentIndex]<this->heap[index])
        {
            swap(this->heap[parentIndex],this->heap[index]);
            index=parentIndex;
        }
        else
        {
            return;
        }

    }
}

template<class DataType>
void MaxHeap<DataType>::swap(Data<DataType>& a,Data<DataType>& b)
{
    Data<DataType> temp=a;
    a=b;
    b=temp;
}

template<class DataType>
void MaxHeap<DataType>::remove()
{
    if(this->size==0)
    {
        cout<<"Heap is empty"<<endl;
        return;
    }
    // 将堆顶元素替换为最后一个元素
    this->heap[0]=this->heap[this->size-1];
    this->size--;

    // 下沉替换后的堆顶元素
    int index=0;
    while(2*index+1<this->size)
    {
        int childIndex=index*2+1;
        if(childIndex+1<this->size&&this->heap[childIndex]<this->heap[childIndex+1])
        {
            childIndex++;
        }
        if(this->heap[index]<this->heap[childIndex])
        {
            swap(this->heap[index],this->heap[childIndex]);
            index=childIndex;
        }
        else 
        {
            return;
        }

    }
}

template<class DataType>
Data<DataType> MaxHeap<DataType>::getMax() const
{
    if(this->size==0)
    {
        throw("Heap is empty");
    }
    return this->heap[0];
}
```

# 实现思路

## 1.用什么结构来实现堆

首先，堆的性质

1. 只能纵向比较，没有横向比较大小
2. 他是完全平衡的

根据性质2，我们可以得到下面结论：他是完全平衡的->每个元素之间是没有空隙的

| 每个节点编号 | 左孩子编号 | 右孩子编号 |
| ------------ | ---------- | ---------- |
| 0            | 1          | 2          |
| 1            | 3          | 4          |
| 2            | 5          | 6          |
| i            | 2*i+1      | 2*i+2      |

判断当前节点是否是分支节点

假设有n个节点，假设当前节点编号是i，那么如果他右孩子至少编号是2*i+1，此时最后一个节点编号是n-1,所以
$$
2*i+1<=n-1
$$
**所以用数组下标来映射节点编号**

## 2.怎么实现

1. 优先级设置

### 增添函数，删除函数思想

1. 增

   在末尾加上元素，然后一步一步向上重建这个堆，只需要与父节点比较，注意这里要交换函数，使用引用

2. 减

   将最后一个元素复制到根，因为这是根的两个子树都是一个堆，所以，只需要向下依次重建就行了

# 易错点

1. 在模版类中实现构造函数，对与模版类型，参数我们可以默认使用这个类型的默认构造函数

   ```c++
    Data(int p=0,DataType d=DataType()):priority(p),data(d){}
   ```

2. 重载=，必须是类的一个非静态成员函数