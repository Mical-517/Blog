---
title: BST Knowledge
published: 2026-02-01T14:19:47+08:00
summary: "二叉排序树&&AVL树涉及知识点导图"
cover:
  image: https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202602011340604.png
tags: [BST]
categories: '知识点'
draft: false
lang: ''
---

# 知识点·

## 一,关于添加元素动态维持树平衡（AVL）

这里设计旋转的概念，利用旋转可以在位置搜索树的前提下，改变树的层级结构，进而维持树的平衡

易错点:

1. 关于改变树的层级结构，**参数要是指针的引用而不是指针**

   因为前者不会创建指向目标节点的副本，而是为原来树结构本身指向改目标节点的指针起一个别名，直接修改这个树；但是后者时创建一个副本，它能够改变他指向节点的成员，但是无法改变指向这个节点的父节点的指向，造成树结构混乱

   

![](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202602011302887.png)

## 二，遍历方式

### 1.递归遍历

略

### 2.非递归遍历

```c++
//使用栈遍历树的三种方式
template<class DataType>
void BSTree<DataType>::preOrderStack()
{
    if(this->root==nullptr) return;
    stack<BSTNode<DataType>*> treeStack;
    treeStack.push(this->root);
    while(!treeStack.empty())
    {
        BSTNode<DataType>* node=treeStack.top();
        treeStack.pop();
        cout<<node->data<<" ";
        if(node->right!=nullptr)
        {
            treeStack.push(node->right);
        }
        if(node->left!=nullptr)
        {
            treeStack.push(node->left);
        }
    }
    cout<<endl;
}

template<class DataType>
void BSTree<DataType>::inOrderStack()
{
    if(this->root==nullptr) return;
    stack<BSTNode<DataType>*> stackTree;
    BSTNode<DataType>* node = this->root;
    while (node != nullptr || !stackTree.empty())
    {
        while (node != nullptr)
        {
            stackTree.push(node);
            node = node->left;
        }
        node = stackTree.top();
        stackTree.pop();
        cout << node->data << " ";
        node = node->right;
    }
    cout<<endl;
}

//后序遍历使用了一个指针来区分是访问当前节点还是将右子树压入栈中
template<class DataType>
void BSTree<DataType>::postOrderStack()
{
    if(this->root==nullptr) return;
    stack<BSTNode<DataType>*> treeStack;
    BSTNode<DataType>* node = this->root;
    BSTNode<DataType>* lastVisited = nullptr;
    while (node != nullptr || !treeStack.empty())
    {
        while (node != nullptr)
        {
            treeStack.push(node);
            node = node->left;
        }
        node = treeStack.top();
        if (node->right == nullptr || node->right == lastVisited)
        {
            cout << node->data << " ";
            treeStack.pop();
            lastVisited = node;
            node = nullptr;
        }
        else
        {
            node = node->right;
        }
    }
    cout<<endl;
}
```

### 3.Morris遍历方法

略

## 三，DSW算法

不同于AVL树动态调整，DSW是一次性平衡整棵树

```c++
//DSW算法将二叉搜索树转换为平衡二叉树,一次性转换
template<class DataType>
void BSTree<DataType>::DSW()
{
    if (this->root == nullptr) return;

    // 1. 将树转换为右链 (Vine)
    DSWHelper(this->root);

    // 2. 压缩平衡
    int n = this->sum;
    // m 是最大的 2^k - 1，且 m <= n
    int m = 1;
    while (m <= n) {
        m = m * 2 + 1;
    }
    m /= 2;

    // 创建虚拟头节点简化旋转逻辑
    BSTNode<DataType> dummy;
    dummy.right = this->root;

    // 第一阶段：消除多余节点，使其数量达到 m
    BSTNode<DataType>* p = &dummy;
    for (int i = 0; i < n - m; i++)
    {
        turnRR(p->right); // 左旋
        p = p->right;
    }

    // 第二阶段：反复压缩，直到成为平衡树
    while (m > 1)
    {
        m /= 2;
        p = &dummy;
        for (int i = 0; i < m; i++)
        {
            turnRR(p->right); // 左旋
            p = p->right;
        }
    }

    //这一步很关键，不然在平衡后的树就没有用了
    this->root = dummy.right;
}

template<class DataType>
void BSTree<DataType>::DSWHelper(BSTNode<DataType>*& rootPtr)
{
    BSTNode<DataType> dummy;
    dummy.right = rootPtr;
    BSTNode<DataType>* p = &dummy;
    BSTNode<DataType>* q = p->right;

    while (q != nullptr)
    {
        if (q->left != nullptr)
        {
            // 如果有左子树，进行右旋 (turnLL)
            // 旋转后，p->right 会变成原来的 q->left
            turnLL(p->right);
            q = p->right; // 更新 q 指向新的局部根节点
        }
        else
        {
            // 如果没有左子树，向下移动
            p = q;
            q = q->right;
        }
    }
    //只有这一步才能在后续Helper函数中从主链的最上面的节点开始
    rootPtr = dummy.right;
}
```

思路就是：1.将整棵树变为右链。2,针对这个友链，先计算最接近总结点数的满二叉树有多少节点，先旋转减去后剩余的多余节点，然后剩下的满二叉树节点进行旋转

![image-20260201145239164](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202602011452287.png)

优化点：使用了虚拟头结点，很好的简化了两个函数的逻辑

## 四，删除节点

### 1.合并删除

```c++
//删除节点，并且还要调整树的平衡
template<class DataType>
void BSTree<DataType>::eraseByMerge(const DataType& data)
{
    BSTNode<DataType>* node=this->root,* prev=nullptr;
    while(node!=nullptr)
    {
        if(node->data==data) break;
        prev=node;
        if(data<node->data) node=node->left;
        else node=node->right;
    }
    //这里为什么不直接传node还要再次判断传pre
    //这就是引用传递，和值传递的区别，一个是传的副本一个是树结构本身的成员指针
    if(node!=nullptr&&node->data==data)
    {
        if(node==this->root)
        {
            eraseByMergeHelper(this->root);
        }
        else
        {
            if(prev->left==node)
            {
                eraseByMergeHelper(prev->left);
            }
            else
            {
                eraseByMergeHelper(prev->right);
            }
        }
    }
    else
    {
        if(this->root!=nullptr)
        {
            cout<<"data"<<data<<"is not in the tree"<<endl;
        }
        else
        {
            cout<<"the tree is empty"<<endl;
        }
    }
}

template<class DataType>
void BSTree<DataType>::eraseByMergeHelper(BSTNode<DataType>*& rootPtr)
{
    BSTNode<DataType>* node=rootPtr;
    if(node!=nullptr)
    {
        if(node->right==nullptr)
        {
            rootPtr=rootPtr->left;
        }
        else
        {
            if(node->left==nullptr)
            {
                rootPtr=rootPtr->right;
            }
            else
            {
                //找到左子树的最右节点
                node=node->left;
                while(node->right!=nullptr)
                {
                    node=node->right;
                }
                //将待删除节点的右子树挂到左子树的最右节点上
                node->right=rootPtr->right;
                node=rootPtr;
                rootPtr=rootPtr->left;
            }
        }
        delete node;
        this->sum--;
    }
    this->DSW();
}
```

### 拷贝删除

```c++
template<class DataType>
void BSTree<DataType>::eraseByCopy(const DataType& data)
{
    //先找到data对应节点的指针
    BSTNode<DataType>* pre=nullptr;
    BSTNode<DataType>* node=this->root;
    while(node!=nullptr)
    {
        if(node->data==data) break;
        pre=node;
        if(data<node->data) node=node->left;
        else node=node->right;
    }
    if(node!=nullptr&&node->data==data)
    {
        if(node==this->root)
        {
            this->eraseCopyHelper(this->root);
        }
        else
        {
            if(pre->left==node)
            {
                this->eraseCopyHelper(pre->left);
            }
            else
            {
                this->eraseCopyHelper(pre->right);
            }
        }
    }
}

template<class DataType>
void BSTree<DataType>::eraseByCopyHelper(BSTNode<DataType>*& rootPtr)
{
    BSTNode<DataType>* temp=rootPtr;
    BSTNode<DataType>* pre=nullptr;
    if(temp->right==nullptr)
    {
        rootPtr=rootPtr->left;
    }
    else
    {
        if(temp->right==nullptr)
        {
            rootPtr=rootPtr->left;
        }
        else
        {
            //找到左子树的最右节点
            temp=temp->left;
            pre=rootPtr;
            while(temp->right!=nullptr)
            {
                pre=temp;
                temp=temp->right;
            }
            rootPtr->data=temp->data;
            //如果左子树没有右子树
            if(pre==rootPtr)
            {
                pre->left=temp->left;
            }
            else
            {
                pre->right=temp->left;
            }
        }
    }
}
```

