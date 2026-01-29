---
title: DLinklist
published: 2026-01-29T17:09:02+08:00
summary: "双向链表学习笔记"
cover:
  image: https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202601291712327.png
tags: [双向链表]
categories: '数据结构'
draft: false
lang: ''
---

# 代码

```c++
#pragma once
#include <iostream>
using namespace std;

template<class DataType>
class DLinkList;

template<class DataType>
ostream& operator<<(ostream& os,const DLinkList<DataType>& temp);

template<class DataType>
class DataNode
{
    friend class DLinkList<DataType>;
    public:
    DataNode()=default;
    //注意这里节点的是浅拷贝，要搭配new的内存进行使用
    DataNode(const DataType inforTemp,DataNode<DataType>* preTemp=nullptr,DataNode<DataType>* nextTemp=nullptr)
    {
        this->pre=preTemp;
        this->next=nextTemp;
        this->infor=inforTemp;
    }
    private:
    DataNode<DataType>* pre=nullptr;
    DataNode<DataType>* next=nullptr;
    DataType infor;
};

template<class DataType>
class DLinkList
{
    public:
    DLinkList()
    {
        sum=0;
        headPtr=new DataNode<DataType>();
        tailPtr=new DataNode<DataType>();
        headPtr->next=tailPtr;
        tailPtr->pre=headPtr;
    }
    DLinkList(const DLinkList<DataType>& temp);
    ~DLinkList()
    {
        clear();
        delete this->headPtr;
        delete this->tailPtr;
        headPtr=tailPtr=nullptr;
    }

    void addTail(const DataType& data);
    ostream& printDLinkList(ostream& os) const;
    void clear();
    void erase(const DataType& data);
    bool empty() const;
    private:
    DataNode<DataType>* headPtr=nullptr;
    DataNode<DataType>* tailPtr=nullptr;
    int sum=0;
    friend ostream& operator<< <DataType>(ostream& os,const DLinkList<DataType>& temp);    

};


template<class DataType>
void DLinkList<DataType>::addTail(const DataType& data)
{
    DataNode<DataType> *actionPtr=new DataNode<DataType>(data);
    if(actionPtr)
    {
        actionPtr->next=this->tailPtr;
        actionPtr->pre=this->tailPtr->pre;
        this->tailPtr->pre->next=actionPtr;
        this->tailPtr->pre=actionPtr;
        this->sum++;
    }
    else
    {
        cout<<"AddTail error"<<endl;
    }
}

template<class DataType>
ostream& DLinkList<DataType>::printDLinkList(ostream& os) const
{
    DataNode<DataType>* actionPtr=this->headPtr->next;
    while(actionPtr!=this->tailPtr)
    {
        os<<actionPtr->infor<<" ";
        actionPtr=actionPtr->next;
    }
    return os;
}

template<class DataType>
ostream& operator<<(ostream& os,const DLinkList<DataType>& temp) 
{
    return temp.printDLinkList(os);
}

template<class DataType>
DLinkList<DataType>::DLinkList(const DLinkList<DataType>& temp):DLinkList()
{
    if(temp.sum==0) return;
    DataNode<DataType>* tempPtr=temp.headPtr->next;
    while(tempPtr!=temp.tailPtr)
    {
        this->addTail(tempPtr->infor);
        tempPtr=tempPtr->next;
    }
}

template<class DataType>
void DLinkList<DataType>::clear()
{
    DataNode<DataType>* currPtr=this->headPtr->next;
    DataNode<DataType>* nextPtr=nullptr;
    while(currPtr!=this->tailPtr)
    {
        nextPtr=currPtr->next;
        delete currPtr;
        currPtr=nextPtr;
    }
    this->sum=0;
}

template<class DataType>
void DLinkList<DataType>::erase(const DataType& data)
{
    if(this->sum==0) return;
    DataNode<DataType>* actionPtr=this->headPtr->next;
    //循环范围是有限的
    while(actionPtr!=this->tailPtr&&actionPtr->infor!=data)
    {
        actionPtr=actionPtr->next;
    }
    if(actionPtr==this->tailPtr) return;
    actionPtr->pre->next=actionPtr->next;
    actionPtr->next->pre=actionPtr->pre;
    delete actionPtr;
    this->sum--;
}

template<class DataType>
bool DLinkList<DataType>::empty() const
{
    return this->sum==0?true:false;
}
```





# 常见问题

1. **类的拷贝构造函数参数不可以是值传递，只可以是引用传递或者是指针**

![image-20260127144856201](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202601271449252.png)

2. **关于委托构造函数作用以及类内声明，类外定义**

   通用层面：委托函数的核心是**一个函数将部分逻辑委托给另一个函数完成**，目的是复用逻辑、解耦；简化代码逻辑，对与普通函数就是相当于函数内部调用

   **重点：**对于构造函数来说，是不可以在内部再次调用一个构造函数的，但是**可以使用委托构造函数来完成**

   **语法结构**：

   ```c++
       // 普通构造函数：初始化num
       MyClass(int x) : num(x) {
           cout << "普通构造函数调用" << endl;
       }
   
       // 正确的拷贝构造函数：初始化列表中委托给普通构造函数
       MyClass(const MyClass& other) : MyClass(other.num) { // 委托语法，正确！
           cout << "拷贝构造函数调用（正确版）" << endl;
       }
   ```

   ```c++
       // 普通构造函数：初始化num
       MyClass(int x) : num(x) {
           cout << "普通构造函数调用" << endl;
       }
   	//一个类中先正常声明
   	MyClass(const MyClass& other);
   	template<```>
   	//类外定义
   	MyClass(const MyClass& other) : MyClass(other.num, other.str) {
           cout << "委托构造函数（拷贝构造）执行" << endl;
       }
   ```

   

3. **重载<<运算符调用打印函数，有关输出流对象是否const的思考，以及关于声明为友元的时候，要显示的将这个重载函数也变为模版函数**

   首先是重载<<的基本格式

   ```c++
   ostream& operator<<(输出流对象，要输出的对象)
   ```

   当<<函数调用类中的打印函数的时候，如果这个类是一个模板类，那么myclass<int>,myclass<double>,都是不同的类，所以说这每个类的<<函数都是不一样的，所以类的<<函数一定要在前面提前声明为模版函数，这样才能够不同类调用不同<<函数

   并且注意这里<<可以连续调用，所以它的返回ostream& 和参数ostream& 都不是`const`的。

   ![image-20260127165055457](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202601271650511.png)

   注意看这里格式：前面已经声明这是个模板函数，所有后面使用的时候，要加<DataType>

   

   坑点：

   1. 重载<<函数只能在类中声明为友元，否则隐含this，三个参数错误

4. **写函数时要根据传递给函数的参数是否可以是const类型来判断参数类型是否可以是const**

   非常量引用的初始值必须为左值，即当一个函数参数是普通的int或者模版参数时，如果在调用的时候，如果传递的是const或者常量就会出错，下图，一定要是const更普遍

![image-20260127152759124](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202601271527157.png)

5. 注意类型转换，const无法变为普通，这里后面参数要使用普通类型

![image-20260127153202108](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202601271532152.png)

6. **这里，调用<<时，temp调用print但是temp是const所以print必须是const才可以**

![image-20260127154348462](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202601271543505.png)

7. ***时刻注意->和.区分开***

8. **当我们调用类中已有的函数的时候，一定要清楚函数的行为动作都有什么，否则极易出错，比如`addTail`已自增`sum`，但手动又执行`sum++`，导致`sum`计数翻倍**

![image-20260127160812233](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202601271608269.png)

9. 在写函数之前一定先判断这个函数是否是一个常函数

10. **`delete`后置空`headPtr/tailPtr`，避免野指针**

11. 析构函数与clear函数的辨析

    #### 先明确`clear`和析构函数的核心职责差异

    |   函数    |             核心目的             |          内存释放范围          |        执行后对象状态         |
    | :-------: | :------------------------------: | :----------------------------: | :---------------------------: |
    | `clear()` | 清空链表的**业务数据**，重置链表 | 仅释放 “数据节点”（业务数据）  | 对象仍可用（可继续`addTail`） |
    | 析构函数  |   销毁整个对象，结束其生命周期   | 释放**所有**堆内存（包括哨兵） |    对象不可用（生命周期结     |

`clear`的设计目标是 “重置链表” 而非 “销毁对象”：比如你清空链表后，可能还想继续往里面加新元素（复用链表），如果`clear`把哨兵节点也删了，后续调用`addTail`时会访问`headPtr/tailPtr`（此时已是空指针），直接触发崩溃；

析构函数的设计目标是 “彻底清理”：当 DLinkList 对象生命周期结束（比如局部对象出作用域、`delete`动态对象），必须释放该对象**所有**在堆上申请的内存 —— 包括数据节点，也包括头尾哨兵节点，否则会造成**内存泄漏**。

12. **内存安全**

    **使用->判断是否是空访问**

    

