---
title: K DTree
published: 2026-02-08T12:46:29+08:00
summary: "k-d树的设计过程与作用"
cover:
  image: https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202602011340604.png
tags: [树结构]
categories: '数据结构'
draft: false
lang: ''
---

# 代码

```c++
#pragma once
#include <iostream>

template<class DataType>
class KDTree;


//二维K-d树节点类
template<class DataType>
class KDNode
{
    public:
    friend class KDTree<DataType>;
    KDNode():left(nullptr),right(nullptr){}
    KDNode(DataType value1,DataType value2,KDNode<DataType>* l=nullptr,KDNode<DataType>* r=nullptr)
    {
        data[0]=value1;
        data[1]=value2;
        left=l;
        right=r;
    }
    //重载==运算符
    bool operator==(const KDNode<DataType>& other) const
    {
        return (data[0] == other.data[0] && data[1] == other.data[1]);
    }
    private:
    DataType data[2];
    KDNode<DataType>* left;
    KDNode<DataType>* right;
};

template<class DataType>
class KDTree
{
    public:
    KDTree():root(nullptr),sum(0){}
    ~KDTree()
    {
        clear();
    }

    //添加节点
    void insert(DataType& value1,DataType& value2);
    //删除节点
    void erase(DataType& value1,DataType& value2);
    //查找节点
    KDNode<DataType>* search(DataType& value1,DataType& value2);
    //清空树
    void clear();
    //最近邻搜索
    KDNode<DataType>* nearestNeighborSearch(const DataType& value1, const DataType& value2);

    private:
    KDNode<DataType>* root;
    int sum;
    //删除节点辅助函数
    KDNode<DataType>* eraseHelper(KDNode<DataType>* rootPtr, DataType& value1, DataType& value2, int currDi);
    //找到目标维度下的最小值节点
    KDNode<DataType>* findMin(KDNode<DataType>* rootPtr, int targetDi, int currDi);
    //清空树
    void clearHelper(KDNode<DataType>*& rootPtr);
    //最近邻搜索辅助函数
    void nearestNeighborSearchHelper(const KDNode<DataType>* rootPtr, const KDNode<DataType>& targetNode, KDNode<DataType>*& bestNode, double& minDistanceSq, int currDi);
    //计算两点距离的平方
    double distanceSquared(const KDNode<DataType>* n1, const KDNode<DataType>* n2);
};

template<class DataType>
void KDTree<DataType>::insert(DataType& value1,DataType& value2)
{
    KDNode<DataType>* newNode=new KDNode<DataType>(value1,value2);
    KDNode<DataType>* pre=nullptr;
    KDNode<DataType>* curr=this->root;
    int i=0;//比较的维度
    while(curr!=nullptr)
    {
        pre=curr;
        if(curr->data[i]>=newNode->data[i]) curr=curr->left;
        else curr=curr->right;
        i=(i+1)%2;
    }
    if(pre==nullptr) this->root=newNode;
    else
    {
        i=(i+1)%2;
        if(pre->data[i]>=newNode->data[i]) pre->left=newNode;
        else pre->right=newNode;
    }
    this->sum++;
}

//分情况讨论,这里我们将找到目标节点的直接后继，意味着如果没有右子树，我们便将左子树变为右子树
template<class DataType>
void KDTree<DataType>::erase(DataType& value1,DataType& value2)
{
    // 检查节点是否存在
    if (search(value1, value2) != nullptr) {
        root = eraseHelper(root, value1, value2, 0);
        sum--;
    }
}

template<class DataType>
KDNode<DataType>* KDTree<DataType>::eraseHelper(KDNode<DataType>* rootPtr, DataType& value1, DataType& value2, int currDi)
{
    if (rootPtr == nullptr) {
        return nullptr;
    }

    // 检查是否找到了要删除的节点
    if (rootPtr->data[0] == value1 && rootPtr->data[1] == value2) {
        // 情况1: 节点有右子树
        if (rootPtr->right != nullptr) {
            KDNode<DataType>* minNode = findMin(rootPtr->right, currDi, (currDi + 1) % 2);
            rootPtr->data[0] = minNode->data[0];
            rootPtr->data[1] = minNode->data[1];
            rootPtr->right = eraseHelper(rootPtr->right, minNode->data[0], minNode->data[1], (currDi + 1) % 2);
        }
        // 情况2: 节点只有左子树
        else if (rootPtr->left != nullptr) {
            KDNode<DataType>* minNode = findMin(rootPtr->left, currDi, (currDi + 1) % 2);
            rootPtr->data[0] = minNode->data[0];
            rootPtr->data[1] = minNode->data[1];
            // 递归删除左子树中的minNode，然后将更新后的左子树挂到右边
            rootPtr->right = eraseHelper(rootPtr->left, minNode->data[0], minNode->data[1], (currDi + 1) % 2);
            rootPtr->left = nullptr;
        }
        // 情况3: 叶子节点
        else {
            delete rootPtr;
            return nullptr;
        }
        return rootPtr;
    }

    // 递归向下查找
    int nextDi = (currDi + 1) % 2;
    DataType targetData[] = {value1, value2};
    if (targetData[currDi] < rootPtr->data[currDi]) {
        rootPtr->left = eraseHelper(rootPtr->left, value1, value2, nextDi);
    } else {
        rootPtr->right = eraseHelper(rootPtr->right, value1, value2, nextDi);
    }
    return rootPtr;
}


template<class DataType>
KDNode<DataType>* KDTree<DataType>::findMin(KDNode<DataType>* rootPtr, int targetDi, int currDi)
{
    if (rootPtr == nullptr) {
        return nullptr;
    }

    KDNode<DataType>* minNode = rootPtr;
    int nextDi = (currDi + 1) % 2;

    if (currDi == targetDi) {
        // 如果当前维度是目标维度，最小值只可能在左子树
        if (rootPtr->left != nullptr) {
            KDNode<DataType>* leftMin = findMin(rootPtr->left, targetDi, nextDi);
            if (leftMin->data[targetDi] < minNode->data[targetDi]) {
                minNode = leftMin;
            }
        }
    } else {
        // 否则，需要同时检查左右子树
        if (rootPtr->left != nullptr) {
            KDNode<DataType>* leftMin = findMin(rootPtr->left, targetDi, nextDi);
            if (leftMin->data[targetDi] < minNode->data[targetDi]) {
                minNode = leftMin;
            }
        }
        if (rootPtr->right != nullptr) {
            KDNode<DataType>* rightMin = findMin(rootPtr->right, targetDi, nextDi);
            if (rightMin->data[targetDi] < minNode->data[targetDi]) {
                minNode = rightMin;
            }
        }
    }
    return minNode;
}


template<class DataType>
KDNode<DataType>* KDTree<DataType>::search(DataType& value1,DataType& value2)
{
    KDNode<DataType>* curr=this->root;
    int i=0;
    KDNode<DataType> newNode(value1,value2);
    while(curr!=nullptr)
    {
        if(*curr==newNode) return curr;
        if(curr->data[i]>=newNode.data[i]) curr=curr->left;
        else curr=curr->right;
        i=(i+1)%2;
    }
    return nullptr;
}

template<class DataType>
void KDTree<DataType>::clear()
{
    clearHelper(this->root);
    this->root=nullptr;
    this->sum=0;
}

template<class DataType>
void KDTree<DataType>::clearHelper(KDNode<DataType>*& rootPtr)
{
    if(rootPtr==nullptr) return;
    clearHelper(rootPtr->left);
    clearHelper(rootPtr->right);
    delete rootPtr;
    rootPtr=nullptr;
}

template<class DataType>
double KDTree<DataType>::distanceSquared(const KDNode<DataType>* n1, const KDNode<DataType>* n2)
{
    double dx = n1->data[0] - n2->data[0];
    double dy = n1->data[1] - n2->data[1];
    return dx * dx + dy * dy;
}

template<class DataType>
KDNode<DataType>* KDTree<DataType>::nearestNeighborSearch(const DataType& value1, const DataType& value2)
{
    if (this->root == nullptr) {
        return nullptr;
    }
    KDNode<DataType> targetNode(value1, value2);
    KDNode<DataType>* bestNode = nullptr;
    double minDistanceSq = -1.0; // 使用-1.0表示无限大

    nearestNeighborSearchHelper(this->root, targetNode, bestNode, minDistanceSq, 0);
    return bestNode;
}


template<class DataType>
void KDTree<DataType>::nearestNeighborSearchHelper(const KDNode<DataType>* rootPtr, const KDNode<DataType>& targetNode, KDNode<DataType>*& bestNode, double& minDistanceSq, int currDi)
{
    if (rootPtr == nullptr) {
        return;
    }

    // 1. 沿着路径向下搜索
    KDNode<DataType>* searchPath = nullptr;
    KDNode<DataType>* otherPath = nullptr;

    if (targetNode.data[currDi] < rootPtr->data[currDi]) {
        searchPath = rootPtr->left;
        otherPath = rootPtr->right;
    } else {
        searchPath = rootPtr->right;
        otherPath = rootPtr->left;
    }

    // 递归进入更可能包含最近邻的子树
    nearestNeighborSearchHelper(searchPath, targetNode, bestNode, minDistanceSq, (currDi + 1) % 2);

    // 2. 回溯
    // (a) 检查当前节点是否是更近的点
    double currentDistSq = distanceSquared(rootPtr, &targetNode);
    if (bestNode == nullptr || currentDistSq < minDistanceSq) {
        minDistanceSq = currentDistSq;
        bestNode = const_cast<KDNode<DataType>*>(rootPtr);
    }

    // (b) 检查是否需要搜索另一侧的子树
    // 计算目标点到分割超平面的距离的平方
    double distToSplitPlaneSq = targetNode.data[currDi] - rootPtr->data[currDi];
    distToSplitPlaneSq *= distToSplitPlaneSq;

    // 如果以目标点为圆心，minDistance为半径的圆与分割平面相交，则另一侧子树可能存在更近的点
    if (distToSplitPlaneSq < minDistanceSq) {
        nearestNeighborSearchHelper(otherPath, targetNode, bestNode, minDistanceSq, (currDi + 1) % 2);
    }
}



```

# 设计思路

## 概述

kd树是一种分割k维数据空间的数据结构，主要应用于多维空间关键数据的搜索

kd树也是二叉树，是用于分割多维空间的数据结构，所以其每一个节点是一个多维坐标。

即：

**把K-D树看做k维空间中的实例点的集合，树上的分叉相当于k维空间中的超平面，超平面将k维空间中的实例点分成两个集合，两个集合分别用分叉点的左右子树表示，若当前节点的划分维度为d，当前节点在d维的坐标值是x，则当前节点左子树所有点在d维的坐标值均小于x，右子树均大于x**

先来看看kd树的构造：

在构造1维BST树时，一个1维数据根据其与树的根结点和中间结点进行大小比较的结果来决定是划分到左子树还是右子树，同理，我们也可以按照这样的方式，将一个K维数据与Kd-tree的根结点和中间结点进行比较，只不过不是对K维数据进行整体的比较，而是选择某一个维度Di，然后比较两个K维数在该维度Di上的大小关系，即每次选择一个维度Di来对K维数据进行划分，相当于用一个垂直于该维度Di的超平面将K维数据空间一分为二，平面一边的所有K维数据在Di维度上的值小于平面另一边的所有K维数据对应维度上的值。也就是说，我们每选择一个维度进行如上的划分，就会将K维数据空间划分为两个部分，如果我们继续分别对这两个子K维空间进行如上的划分，又会得到新的子空间，对新的子空间又继续划分，重复以上过程直到每个子空间都不能再划分为止。以上就是构造Kd-Tree的过程，上述过程中涉及到两个重要步骤：

1. 划分时以选中维度的中位数来划分，小的为左节点，大的为右节点。

2. 选择维度的方法有最大方差法或顺序遍历法，这里采用顺序遍历法，即对于点（x1, x2, x3, x4 ....），第一次按x1的维度来切分，第二次按照x2的维度来切分，切刀最后一个维度之后又回到x1的维度。

来看看一个二维的例子：

这是一个有13个点的二维图

![image-20260208125148970](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202602081251021.png)



![image-20260208125204876](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202602081252954.png)



首先先以 x 坐标进行切分，我们选出 x 坐标的中位点，获取最根部节点的坐标（7， 2）

之后，左右两边再按照 y 轴的排序进行切分，中位点记载于左右枝的节点。左边区域按(5, 4)上下分割，右边区域按（9，6）上下分割。

![image-20260208125218390](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202602081252439.png)



在不断按照上面的步骤进行切分和构造，得到最后结果：

![image-20260208125230011](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202602081252083.png)



![image-20260208125245087](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202602081252153.png)

## 主要代码

### 插入函数实现

在一个k-d树上插入，关键就是每一层要比较的关键字不一样，所以我们将每一个关键字与一个数字关联起来，然后对数字+1取余来进行循环

再次基础上，**我们要再次回顾有关指针的引用与指针复制的区别所在，我们要改变树的结构，如果相关函数没有返回指针，并且是迭代进行指针指向改变的时候，我们就要使用指针的引用了**

### 删除节点函数（比较复杂）

对与k-d树的删除节点我们有几方面的考虑

1. 每一层关键字的比较
2. 待删除节点的后继与前驱节点的位置与BST树不同，因为每一层比较的关键字不同
3. 待删除节点的后代情况复杂

思路：

我们很容易想到可以利用search函数来找到对应的待删除的节点，但是相应的我们返回的是一个指向待删除节点的指针，并不是父节点的成员指针，改变他不会改变树本身的结构。所以search函数只用来帮助我们判断待删除节点是否存在的问题

当节点存在时：调用删除函数，从根节点开始扫描迭代进行，如果当前节点是待删除节点，就找到他的直接后继

这里关于直接后继的找法：因为不是BST，所以

1. 如果有左右子树，左右子树在待删除节点的比较维度下都可能有节点比待删除节点小，所以要扫描左右子树
2. 如果没有子树，直接删除
3. 如果只有右子树，刚好就在右子树里面搜索最小的
4. 如果只有左子树，就要将左子树变为右子树然后找右子树的最小的

然后将找到的后继类似复制删除，复制上去，**再次调用删除函数将下层复制的那个节点再次删除**

**体会函数参数的选取：**

删除函数是从根节点开始扫描，所以每个函数参数表示的是当前节点的比较维度

但是findmin函数要找到待删除节点维度下的直接后继，所以有两个参数

### 最近邻点搜索函数

知识概要简介：

[KNN(K近邻)算法之——KD-Tree构建及查找原理 - hello_nullptr - 博客园](https://www.cnblogs.com/hello-nullptr/p/18369092)

关键函数实现:

1. 没有返回值类型

   ```c++
   // ...existing code...
       //清空树
       void clearHelper(KDNode<DataType>*& rootPtr);
       //最近邻搜索辅助函数
       void nearestNeighborSearchHelper(const KDNode<DataType>* rootPtr, const KDNode<DataType>& targetNode, KDNode<DataType>*& bestNode, double& minDistanceSq, int currDi);
       //计算两点距离的平方
       double distanceSquared(const KDNode<DataType>* n1, const KDNode<DataType>* n2);
   };
   
   template<class DataType>
   void KDTree<DataType>::insert(DataType& value1,DataType& value2)
   // ...existing code...
   // ...existing code...
   template<class DataType>
   void KDTree<DataType>::clearHelper(KDNode<DataType>*& rootPtr)
   {
   // ...existing code...
       delete rootPtr;
       rootPtr=nullptr;
   }
   
   template<class DataType>
   double KDTree<DataType>::distanceSquared(const KDNode<DataType>* n1, const KDNode<DataType>* n2)
   {
       double dx = n1->data[0] - n2->data[0];
       double dy = n1->data[1] - n2->data[1];
       return dx * dx + dy * dy;
   }
   
   template<class DataType>
   KDNode<DataType>* KDTree<DataType>::nearestNeighborSearch(const DataType& value1, const DataType& value2)
   {
       if (this->root == nullptr) {
           return nullptr;
       }
       KDNode<DataType> targetNode(value1, value2);
       KDNode<DataType>* bestNode = nullptr;
       double minDistanceSq = -1.0; // 使用-1.0表示无限大
   
       nearestNeighborSearchHelper(this->root, targetNode, bestNode, minDistanceSq, 0);
       return bestNode;
   }
   
   template<class DataType>
   void KDTree<DataType>::nearestNeighborSearchHelper(const KDNode<DataType>* rootPtr, const KDNode<DataType>& targetNode, KDNode<DataType>*& bestNode, double& minDistanceSq, int currDi)
   {
       if (rootPtr == nullptr) {
           return;
       }
   
       // 1. 沿着路径向下搜索
       KDNode<DataType>* searchPath = nullptr;
       KDNode<DataType>* otherPath = nullptr;
   
       if (targetNode.data[currDi] < rootPtr->data[currDi]) {
           searchPath = rootPtr->left;
           otherPath = rootPtr->right;
       } else {
           searchPath = rootPtr->right;
           otherPath = rootPtr->left;
       }
   
       // 递归进入更可能包含最近邻的子树
       nearestNeighborSearchHelper(searchPath, targetNode, bestNode, minDistanceSq, (currDi + 1) % 2);
   
       // 2. 回溯
       // (a) 检查当前节点是否是更近的点
       double currentDistSq = distanceSquared(rootPtr, &targetNode);
       if (bestNode == nullptr || currentDistSq < minDistanceSq) {
           minDistanceSq = currentDistSq;
           bestNode = const_cast<KDNode<DataType>*>(rootPtr);
       }
   
       // (b) 检查是否需要搜索另一侧的子树
       // 计算目标点到分割超平面的距离的平方
       double distToSplitPlaneSq = targetNode.data[currDi] - rootPtr->data[currDi];
       distToSplitPlaneSq *= distToSplitPlaneSq;
   
       // 如果以目标点为圆心，minDistance为半径的圆与分割平面相交，则另一侧子树可能存在更近的点
       if (distToSplitPlaneSq < minDistanceSq) {
           nearestNeighborSearchHelper(otherPath, targetNode, bestNode, minDistanceSq, (currDi + 1) % 2);
       }
   }
   ```

2. 带返回值类型

   ```c++
   // ...existing code...
       //清空树
       void clearHelper(KDNode<DataType>*& rootPtr);
       //最近邻搜索辅助函数
       KDNode<DataType>* nearestNeighborSearchHelper(const KDNode<DataType>* rootPtr, const KDNode<DataType>& targetNode, double& minDistanceSq, int currDi);
       //计算两点距离的平方
       double distanceSquared(const KDNode<DataType>* n1, const KDNode<DataType>* n2);
   };
   
   // ...existing code...
   template<class DataType>
   void KDTree<DataType>::clearHelper(KDNode<DataType>*& rootPtr)
   {
   // ...existing code...
       delete rootPtr;
       rootPtr=nullptr;
   }
   
   template<class DataType>
   double KDTree<DataType>::distanceSquared(const KDNode<DataType>* n1, const KDNode<DataType>* n2)
   {
       double dx = n1->data[0] - n2->data[0];
       double dy = n1->data[1] - n2->data[1];
       return dx * dx + dy * dy;
   }
   
   template<class DataType>
   KDNode<DataType>* KDTree<DataType>::nearestNeighborSearch(const DataType& value1, const DataType& value2)
   {
       if (this->root == nullptr) {
           return nullptr;
       }
       KDNode<DataType> targetNode(value1, value2);
       double minDistanceSq = -1.0; // 使用-1.0或一个很大的数表示无限大
   
       return nearestNeighborSearchHelper(this->root, targetNode, minDistanceSq, 0);
   }
   
   
   template<class DataType>
   KDNode<DataType>* KDTree<DataType>::nearestNeighborSearchHelper(const KDNode<DataType>* rootPtr, const KDNode<DataType>& targetNode, double& minDistanceSq, int currDi)
   {
       if (rootPtr == nullptr) {
           return nullptr;
       }
   
       // 1. 沿着路径向下搜索
       const KDNode<DataType>* searchPath = nullptr;
       const KDNode<DataType>* otherPath = nullptr;
   
       if (targetNode.data[currDi] < rootPtr->data[currDi]) {
           searchPath = rootPtr->left;
           otherPath = rootPtr->right;
       } else {
           searchPath = rootPtr->right;
           otherPath = rootPtr->left;
       }
   
       // 递归进入更可能包含最近邻的子树，并获取该子树中找到的最佳节点
       KDNode<DataType>* bestNode = nearestNeighborSearchHelper(searchPath, targetNode, minDistanceSq, (currDi + 1) % 2);
   
       // 2. 回溯
       // (a) 检查当前节点是否是更近的点
       double currentDistSq = distanceSquared(rootPtr, &targetNode);
       if (bestNode == nullptr || currentDistSq < minDistanceSq) {
           minDistanceSq = currentDistSq;
           bestNode = const_cast<KDNode<DataType>*>(rootPtr);
       }
   
       // (b) 检查是否需要搜索另一侧的子树
       double distToSplitPlaneSq = targetNode.data[currDi] - rootPtr->data[currDi];
       distToSplitPlaneSq *= distToSplitPlaneSq;
   
       if (distToSplitPlaneSq < minDistanceSq) {
           // 在另一侧子树中搜索，并与当前已知的最佳节点比较
           KDNode<DataType>* otherBest = nearestNeighborSearchHelper(otherPath, targetNode, minDistanceSq, (currDi + 1) % 2);
           if (otherBest != nullptr) {
               // minDistanceSq 已经在递归调用中被更新，我们只需更新 bestNode
               bestNode = otherBest;
           }
       }
       
       return bestNode;
   }
   ```

关键要点：

1. 递归向下搜索，然后回溯检查的整体思想

2. 在回溯检查的过程中，**重要的依旧就是参数中的mindistance，他是全局的一个判断变量，用引用类型**

3. 再进行另一侧子树的检查的时候，一开始我认为检查另一侧一定行不通，因为我一开始想的是既然要查的点在searchPath下，那么他就肯定不在otherPath下，所以再次调用的时候，她连一开始向下递归搜索都做不到

   但是，实际是我忽略了这个点的维度是相对的，比如目标节点在当前节点上方，那他就不可以与当前节点下方的节点比较了吗，当然是可以的，因为可以与下方节点左右比较，上下只是当前节点的比较，不影响其他节点。







------

版权声明：本文为CSDN博主「学习的西瓜皮」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_39747794/article/details/82262576