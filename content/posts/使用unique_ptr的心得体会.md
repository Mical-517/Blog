---
title: 使用unique_ptr的心得体会
published: 2026-03-07T18:08:06+08:00
summary: "使用智能指针重构了BST树"
cover:
  image: https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202602011340604.png
tags: [unique_ptr]
categories: 'c++新特性'
draft: false
lang: ''
---



# 总结

对比裸指针来说，思考智能指针的时候是要从**所有权的角度来分析的，因为智能指针的特点就是当移动语义复制的时候，智能指针原来的指向的堆内存就会被释放，所以，要么我这个新的智能指针本来就没有指向堆内存，然后对其移动语义复制，要么是要先把他掌控的那块堆内存重新让给一个新的智能指针，然后在接受其他智能指针的移动语义复制**

案例：

1. 新的智能指针本来就没有指向堆内存

   ```c++
   // 拷贝构造函数
   template <class DataType>
   DLinkList<DataType>::DLinkList(const DLinkList &other) : DLinkList()
   {
       // 深拷贝
       // 行动指针遍历other,应该用普通指针遍历，因为智能指针遍历是会释放空间
       Node<DataType> *otherPtr = other.head->next.get();
       Node<DataType> *currPtr = this->head.get();
       while (otherPtr != other.tail)
       {
           // 创建新节点，将当前节点的下个节点的所有权交给当前节点的next
           auto newNode = make_unique<Node<DataType>>(otherPtr->infor, currPtr, move(currPtr->next));
           tail->pre = newNode.get();
           // 转移当前节点所有权
           currPtr->next = move(newNode);
           // 更新行动指针
           otherPtr = otherPtr->next.get();
           currPtr = currPtr->next.get();
       }
   }
   ```

   按照原来的裸指针逻辑，插入的时候是

   ```c++
   auto newNode=new Data(data,this->head,head->next);//这是直接插入在原来的头节点后面了
   ```

   但是按照所有权的逻辑，应该是新节点的next（智能指针）接受head->next指向的这块内存的所有权，然后本身节点的堆内存所有权变为head->next（智能指针）所有。

2. 要么是要先把他掌控的那块堆内存重新让给一个新的智能指针，然后在接受其他智能指针的移动语义复制

   ```c++
   // 旋转
   // LL型,RR,LR,RL
   template <class DataType>
   void BSTree<DataType>::turnLL(unique_ptr<Node<DataType>> &grandPtr)
   {
       // 先处理节点孩子所有权
       auto gleft = move(grandPtr->left);
       grandPtr->left = move(gleft->right);
       gleft->right = move(grandPtr);
       grandPtr = move(gleft);
   }
   ```

   就是先用新的智能指针gleft先保存所有权，避免后续grandPtr->left=····,grandleft->left指向的内存被释放，后面在将新指针的所有权转让，类似倒水的问题。



# BST树重构

```c++
#include <iostream>
#include <memory>
using namespace std;

template <class DataType>
class BSTree;

template <class DataType>
class Node
{
public:
    Node() = default;
    Node(const DataType &infor, unique_ptr<Node<DataType>> left, unique_ptr<Node<DataType>> right)
    {
        this->infor = infor;
        this->left = move(left);
        this->right = move(right);
    }

private:
    DataType infor;
    unique_ptr<Node<DataType>> left;
    unique_ptr<Node<DataType>> right;

    friend class BSTree<DataType>;
};

template <class DataType>
class BSTree
{
public:
    BSTree() = default;
    // 插入元素
    void insert_interface(const DataType &data);
    // 前序遍历
    void preOrder();

private:
    unique_ptr<Node<DataType>> root;
    int size{0};

    // 插入辅助函数
    void insertHelper(unique_ptr<Node<DataType>> &ptr, const DataType &data);
    // 得到节点树的平衡度
    int getBalance(const unique_ptr<Node<DataType>> &ptr);
    // 得到节点树的高度
    int getHeight(const unique_ptr<Node<DataType>> &ptr);
    // LL型,RR,LR,RL
    void turnLL(unique_ptr<Node<DataType>> &grandPtr);
    void turnRR(unique_ptr<Node<DataType>> &grandPtr);
    void turnLR(unique_ptr<Node<DataType>> &grandPtr);
    void turnRL(unique_ptr<Node<DataType>> &grandPtr);
    // 前序遍历
    void preOrderHelper(const unique_ptr<Node<DataType>> &ptr);
};

// 插入元素
template <class DataType>
void BSTree<DataType>::insert_interface(const DataType &data)
{
    if (this->root == nullptr)
    {
        this->root = make_unique<Node<DataType>>(data, nullptr, nullptr);
        return;
    }
    this->insertHelper(this->root, data);
}

// 插入元素辅助函数
template <class DataType>
void BSTree<DataType>::insertHelper(unique_ptr<Node<DataType>> &ptr, const DataType &data)
{
    if (ptr == nullptr)
    {
        ptr = make_unique<Node<DataType>>(data, nullptr, nullptr);
        this->size++;
        return;
    }
    // 判断分支
    if (data <= ptr->infor)
    {
        insertHelper(ptr->left, data);
    }
    else
    {
        insertHelper(ptr->right, data);
    }
    // AVL动态平衡树
    // 1.得到当前节点数的平衡度
    int balance = getBalance(ptr);
    if (balance > 1)
    {
        if (getBalance(ptr->left) == 1) // LL
        {
            turnLL(ptr);
        }
        else // LR
        {
            turnLR(ptr);
        }
    }
    if (balance < -1)
    {
        if (getBalance(ptr->right) == -1) // RR
        {
            turnRR(ptr);
        }
        else // RL
        {
            turnRL(ptr);
        }
    }
}

template <class DataType>
int BSTree<DataType>::getBalance(const unique_ptr<Node<DataType>> &ptr)
{
    if (ptr == nullptr)
        return 0;
    int leftTree = getHeight(ptr->left);
    int rightTree = getHeight(ptr->right);
    return leftTree - rightTree;
}

template <class DataType>
int BSTree<DataType>::getHeight(const unique_ptr<Node<DataType>> &ptr)
{
    if (ptr == nullptr)
        return 0;
    int left = getHeight(ptr->left);
    int right = getHeight(ptr->right);
    return left > right ? left + 1 : right + 1;
}

// 旋转
// LL型,RR,LR,RL
template <class DataType>
void BSTree<DataType>::turnLL(unique_ptr<Node<DataType>> &grandPtr)
{
    // 先处理节点孩子所有权
    auto gleft = move(grandPtr->left);
    grandPtr->left = move(gleft->right);
    gleft->right = move(grandPtr);
    grandPtr = move(gleft);
}

template <class DataType>
void BSTree<DataType>::turnRR(unique_ptr<Node<DataType>> &grandPtr)
{
    auto gright=move(grandPtr->right);
    grandPtr->right=move(gright->left);
    gright->left=move(grandPtr);
    grandPtr=move(gright);
}

template <class DataType>
void BSTree<DataType>::turnLR(unique_ptr<Node<DataType>> &grandPtr)
{
    turnRR(grandPtr->left);
    turnLL(grandPtr);
}

template <class DataType>
void BSTree<DataType>::turnRL(unique_ptr<Node<DataType>> &grandPtr)
{
    turnLL(grandPtr->right);
    turnRR(grandPtr);
}



// 前序遍历
template <class DataType>
void BSTree<DataType>::preOrderHelper(const unique_ptr<Node<DataType>> &ptr)
{
    if (ptr == nullptr)
        return;
    cout << ptr->infor << " ";
    preOrderHelper(ptr->left);
    preOrderHelper(ptr->right);
}
template <class DataType>
void BSTree<DataType>::preOrder()
{
    if (this->root == nullptr)
        return;
    preOrderHelper(this->root);
}

```

# 双向链表重构

```c++
#pragma once
#include <memory>
using namespace std;

template <class DataType>
class DLinkList;

template <class DataType>
class Node
{
public:
    Node() = default;
    Node(const DataType &infor, Node<DataType> *pre, std::unique_ptr<Node<DataType>> next)
    {
        this->infor = infor;
        this->pre = pre;
        this->next = move(next);
    }

private:
    // 受到智能指针特性的限制，一块堆内存智能有一个智能指针管理，所以node成员有一个智能指针用于管理，一个裸指针用于观测
    DataType infor;
    std::unique_ptr<Node<DataType>> next;
    Node<DataType> *pre{nullptr};

    friend class DLinkList<DataType>;
};

template <class DataType>
class DLinkList
{
public:
    // 默认构造函数
    DLinkList()
    {
        this->head = make_unique<Node<DataType>>();
        this->head->next = make_unique<Node<DataType>>();
        this->tail = this->head->next.get();
        tail->pre = this->head.get();
    }
    // 拷贝构造函数
    DLinkList(const DLinkList &other);
    // 析构函数
    ~DLinkList() = default;
    // 前插入
    void push_front(const DataType &data);
    // 后插入
    void push_back(const DataType &data);
    // 打印所有数据
    void print();

private:
    std::unique_ptr<Node<DataType>> head;
    Node<DataType> *tail{nullptr}; // 仅用来观测
};

// 拷贝构造函数
template <class DataType>
DLinkList<DataType>::DLinkList(const DLinkList &other) : DLinkList()
{
    // 深拷贝
    // 行动指针遍历other,应该用普通指针遍历，因为智能指针遍历是会释放空间
    Node<DataType> *otherPtr = other.head->next.get();
    Node<DataType> *currPtr = this->head.get();
    while (otherPtr != other.tail)
    {
        // 创建新节点，将当前节点的下个节点的所有权交给当前节点的next
        auto newNode = make_unique<Node<DataType>>(otherPtr->infor, currPtr, move(currPtr->next));
        tail->pre = newNode.get();
        // 转移当前节点所有权
        currPtr->next = move(newNode);
        // 更新行动指针
        otherPtr = otherPtr->next.get();
        currPtr = currPtr->next.get();
    }
}

// 前插入
template <class DataType>
void DLinkList<DataType>::push_front(const DataType &data)
{
    auto newNode = make_unique<Node<DataType>>(data, this->head.get(), move(head->next));
    newNode->next->pre = newNode.get();
    head->next = move(newNode);
}

// 后插入
template <class DataType>
void DLinkList<DataType>::push_back(const DataType &data)
{
    auto ptr = make_unique<Node<DataType>>(data, this->tail->pre, move(this->tail->pre->next));
    this->tail->pre = ptr.get();
    ptr->pre->next = move(ptr);
}

// 打印
template <class DataType>
void DLinkList<DataType>::print()
{
    auto ptr = this->head->next.get();
    while (ptr != this->tail)
    {
        cout << ptr->infor << " ";
        ptr = ptr->next.get();
    }
}

```

