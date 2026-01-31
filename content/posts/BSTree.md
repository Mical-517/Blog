---
title: BSTree
published: 2026-01-31T23:41:38+08:00
summary: "AVL树实现"
cover:
  image: https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202601300127015.png
tags: [树结构]
categories: '数据结构'
draft: false
lang: ''
---

```c++
#pragma once 
#include <iostream>
#include <stack>
#include <queue>
using namespace std;

template<class DataType>
class BSTree;

//树节点
template <class DataType>
class BSTNode
{
    public:
    BSTNode():left(nullptr),right(nullptr){}
    BSTNode(const DataType& data,BSTNode<DataType>* left=nullptr,BSTNode<DataType>* right=nullptr)
    {
        this->data=data;
        this->left=left;
        this->right=right;
    }
    bool operator==(const BSTNode<DataType>& other)
    {
        return this->data==other.data&&this->left==other.left&&this->right==other.right;
    }
    private:
    DataType data;
    BSTNode<DataType>* left;
    BSTNode<DataType>* right;

    friend class BSTree<DataType>;
};

//二叉搜索树
template <class DataType>
class BSTree
{
    public:
    BSTree()=default;
    ~BSTree()
    {
        clear();
    }

    //添加节点
    void insert(const DataType& data);
    //删除节点
    void erase(const DataType& data);
    //递归遍历树的三种遍历方式
    void preOrder();
    void inOrder();
    void postOrder();
    //使用栈遍历树的三种方式
    void preOrderStack();
    void inOrderStack();
    void postOrderStack();
    //广度遍历树
    void levelOrder();
    //清空
    void clear();

    private:
    BSTNode<DataType>* root=nullptr;
    int sum=0;

    //计算节点的度
    int getBalance(BSTNode<DataType>* node);
    //旋转节点
    void turnLL(BSTNode<DataType>*& node);
    void turnRR(BSTNode<DataType>*& node);
    void turnLR(BSTNode<DataType>*& node);
    void turnRL(BSTNode<DataType>*& node);

    //清空树辅助函数
    void clearHelper(BSTNode<DataType>* rootPtr);
    //添加节点辅助函数
    void insertHelper(BSTNode<DataType>*& rootPtr,const DataType& data);
    //递归遍历辅助函数
    void preOrderHelper(BSTNode<DataType>* rootPtr);
    void inOrderHelper(BSTNode<DataType>* rootPtr);
    void postOrderHelper(BSTNode<DataType>* rootPtr);
    //广度遍历辅助函数
    void levelOrderHelper(BSTNode<DataType>* rootPtr);
};



// 递归遍历实现
template<class DataType>
void BSTree<DataType>::preOrder()
{
    preOrderHelper(this->root);
}

template<class DataType>
void BSTree<DataType>::preOrderHelper(BSTNode<DataType>* rootPtr)
{
    if(rootPtr==nullptr) return;
    cout<<rootPtr->data<<endl;
    preOrderHelper(rootPtr->left);
    preOrderHelper(rootPtr->right);
}

template<class DataType>
void BSTree<DataType>::inOrder()
{
    inOrderHelper(this->root);
}

template<class DataType>
void BSTree<DataType>::inOrderHelper(BSTNode<DataType>* rootPtr)
{
    if(rootPtr==nullptr) return;
    inOrderHelper(rootPtr->left);
    cout<<rootPtr->data<<endl;
    inOrderHelper(rootPtr->right);
}

template<class DataType>
void BSTree<DataType>::postOrder()
{
    postOrderHelper(this->root);
}

template<class DataType>
void BSTree<DataType>::postOrderHelper(BSTNode<DataType>* rootPtr)
{
    if(rootPtr==nullptr) return;
    postOrderHelper(rootPtr->left);
    postOrderHelper(rootPtr->right);
    cout<<rootPtr->data<<endl;
}

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
        cout<<node->data<<endl;
        if(node->right!=nullptr)
        {
            treeStack.push(node->right);
        }
        if(node->left!=nullptr)
        {
            treeStack.push(node->left);
        }
    }
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
        cout << node->data << endl;
        node = node->right;
    }
}

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
            cout << node->data << endl;
            treeStack.pop();
            lastVisited = node;
            node = nullptr;
        }
        else
        {
            node = node->right;
        }
    }
}

//添加节点，实现动态调整树的平衡
template<class DataType>
void BSTree<DataType>::insertHelper(BSTNode<DataType>*& rootPtr,const DataType& data)
{
    if(rootPtr==nullptr)
    {
        rootPtr=new BSTNode<DataType>(data);
        this->sum++;
        return;
    }
    if(data>=rootPtr->data)
    {
        insertHelper(rootPtr->right,data);
    }
    else
    {
        insertHelper(rootPtr->left,data);
    }
    // 使用 AVL 树通过检查节点的平衡因子来判断是否需要旋转
    int balance = getBalance(rootPtr->left) - getBalance(rootPtr->right);

    if (balance > 1)
    {
        if (getBalance(rootPtr->left->left) >= getBalance(rootPtr->left->right))
        {
            // LL 型，进行右旋（代码中的 turnLL）
            turnLL(rootPtr);
        }
        else
        {
            // LR 型，先左旋后右旋（代码中的 turnLR）
            turnLR(rootPtr);
        }
    }
    else if (balance < -1)
    {
        if (getBalance(rootPtr->right->right) >= getBalance(rootPtr->right->left))
        {
            // RR 型，进行左旋（代码中的 turnRR）
            turnRR(rootPtr);
        }
        else
        {
            // RL 型，先右旋后左旋（代码中的 turnRL）
            turnRL(rootPtr);
        }
    }

}

template<class DataType>
void BSTree<DataType>::insert(const DataType& data)
{
    insertHelper(this->root,data);
}

template<class DataType>
void BSTree<DataType>::turnRR(BSTNode<DataType>*& node)
{
    BSTNode<DataType>* rightNode = node->right;
    node->right = rightNode->left;
    rightNode->left = node;
    node = rightNode;
}

template<class DataType>
void BSTree<DataType>::turnLL(BSTNode<DataType>*& node)
{
    BSTNode<DataType>* leftNode = node->left;
    node->left = leftNode->right;
    leftNode->right = node;
    node = leftNode;
}

template<class DataType>
void BSTree<DataType>::turnLR(BSTNode<DataType>*& node)
{
    turnRR(node->left);
    turnLL(node);
}

template<class DataType>
void BSTree<DataType>::turnRL(BSTNode<DataType>*& node)
{
    turnLL(node->right);
    turnRR(node);
}

//清空函数
template<class DataType>
void BSTree<DataType>::clear()
{
    this->clearHelper(this->root);
    this->root=nullptr;
    this->sum=0;
}

//清空树辅助函数
template<class DataType>
void BSTree<DataType>::clearHelper(BSTNode<DataType>* rootPtr)
{
    if(rootPtr==nullptr) return;
    clearHelper(rootPtr->left);
    clearHelper(rootPtr->right);
    delete rootPtr;
}

template<class DataType>
int BSTree<DataType>::getBalance(BSTNode<DataType>* node)
{
    if(node==nullptr) return 0;
    int leftHeight=getBalance(node->left);
    int rightHeight=getBalance(node->right);
    return leftHeight>rightHeight?leftHeight+1:rightHeight+1;
}

template<class DataType>
void BSTree<DataType>::levelOrder()
{
    if(this->root==nullptr) return;
    levelOrderHelper(this->root);
}

template<class DataType>
void BSTree<DataType>::levelOrderHelper(BSTNode<DataType>* rootPtr)
{
    queue<BSTNode<DataType>*> treeQueue;
    treeQueue.push(rootPtr);
    while(!treeQueue.empty())
    {
        BSTNode<DataType>* node = treeQueue.front();
        treeQueue.pop();
        cout<<node->data<<endl;
        if(node->left!=nullptr) treeQueue.push(node->left);
        if(node->right!=nullptr) treeQueue.push(node->right);
    }
}
```

代码是一个**模板化的 AVL 树（自平衡二叉搜索树）**

#### 架构核心特点

1. **分层设计**：

   - 底层：`BSTNode` 节点类（封装节点数据、左右孩子指针）；
   - 上层：`BSTree` 树类（封装树的核心操作，通过友元访问节点私有成员）。

   

2. **模板化**：

   - 所有类都基于 `template<class DataType>` 实现，支持任意可比较的类型（如 `int`、`string` 等），复用性极强。

   

3. **接口分离**：

   - 公有接口（对外暴露的功能：插入、遍历、清空等）；
   - 私有辅助函数（内部实现细节：递归辅助、旋转、平衡计算等），符合 “最小暴露原则”。

   

4. **内存安全**：

   - 析构函数调用 `clear()` 释放所有节点内存，避免内存泄漏；
   - `clear()` 递归释放每个节点，最后重置根节点和节点数。

   

### 一、核心功能模块详解

#### 1. 节点类（BSTNode）：基础数据单元

|      功能      |                           实现说明                           |
| :------------: | :----------------------------------------------------------: |
|    构造函数    | ① 无参构造：左右孩子默认空；② 带参构造：初始化数据、左右孩子 |
| 相等运算符重载 |     比较两个节点的 `data`、`left`、`right` 是否完全一致      |
|     封装性     |  成员变量（data/left/right）私有，仅通过友元 `BSTree` 访问   |

#### 2. 核心操作模块：树的增 / 清 / 析构

|      功能      |                           实现说明                           |
| :------------: | :----------------------------------------------------------: |
| 插入（insert） | ① 调用 `insertHelper` 递归插入节点；② 插入后计算平衡因子，判断是否失衡；③ 失衡时执行对应旋转（LL/LR/RR/RL），保证 AVL 树的平衡（左右子树高度差≤1）；④ 节点数 `sum` 自增 |
| 清空（clear）  | ① 调用 `clearHelper` 后序递归释放所有节点内存；② 重置 `root=nullptr`、`sum=0` |
|    析构函数    |        自动调用 `clear()`，确保对象销毁时释放所有内存        |
| 删除（erase）  | 仅声明未实现（代码里只有 `void erase(const DataType& data);`，无具体逻辑） |

#### 3. 遍历模块：递归 + 非递归 + 层序（全覆盖）

|     遍历方式     |               实现方式                |                           核心逻辑                           |
| :--------------: | :-----------------------------------: | :----------------------------------------------------------: |
|     递归前序     |   `preOrder()` + `preOrderHelper()`   |                         根 → 左 → 右                         |
|     递归中序     |    `inOrder()` + `inOrderHelper()`    |           左 → 根 → 右（二叉搜索树中序遍历为升序）           |
|     递归后序     |  `postOrder()` + `postOrderHelper()`  |                         左 → 右 → 根                         |
| 非递归前序（栈） |           `preOrderStack()`           | ① 根节点入栈；② 出栈打印，先压右孩子、再压左孩子（栈先进后出） |
| 非递归中序（栈） |           `inOrderStack()`            |         ① 一路压左孩子入栈；② 出栈打印，再处理右孩子         |
| 非递归后序（栈） |          `postOrderStack()`           | ① 用 `lastVisited` 标记已访问的右孩子；② 确保右孩子访问后再打印当前节点 |
| 层序遍历（队列） | `levelOrder()` + `levelOrderHelper()` | ① 根节点入队；② 出队打印，依次压入左、右孩子（队列先进先出） |

#### 4. AVL 平衡调整模块：核心自平衡逻辑

AVL 树的核心是 “插入后保持平衡”，代码通过以下函数实现：

|      函数      |                             功能                             |                           核心逻辑                           |
| :------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| `getBalance()` | 计算节点高度（注：代码中命名易混淆，实际是**高度**，平衡因子 = 左子树高度 - 右子树高度） |  递归计算左右子树高度，返回当前节点的高度（空节点高度为 0）  |
|   `turnLL()`   |                   LL 型失衡（左左）→ 右旋                    | ① 取当前节点的左孩子为新根；② 原左孩子的右子树挂到当前节点的左；③ 当前节点挂到新根的右 |
|   `turnRR()`   |                   RR 型失衡（右右）→ 左旋                    | ① 取当前节点的右孩子为新根；② 原右孩子的左子树挂到当前节点的右；③ 当前节点挂到新根的左 |
|   `turnLR()`   |               LR 型失衡（左右）→ 先左旋后右旋                |  先对当前节点的左孩子执行 RR 旋转，再对当前节点执行 LL 旋转  |
|   `turnRL()`   |               RL 型失衡（右左）→ 先右旋后左旋                |  先对当前节点的右孩子执行 LL 旋转，再对当前节点执行 RR 旋转  |

### 二，常见错误易错点

#### 1. 栈类型不匹配（对象栈 vs 指针栈）

- 错误代码： 你在 preOrderStack 等函数中定义了 stack<BSTNode<DataType>> （存储的是 对象 ）。
- 报错原因：
  - 当你执行 treeStack.push(*rootPtr) 时，会触发 对象拷贝 。二叉树节点通常包含指针，直接拷贝对象而不处理深拷贝会导致多个节点指向相同的子节点，引发内存问题。
  - 在 postOrderStack 中，你尝试将 node.left （一个指针）推入一个存储对象的栈，编译器无法自动将 BSTNode* 转换为 BSTNode 对象，因此报出 invalid user-defined conversion （非法转换）。
- 解决方法： 将栈改为 stack<BSTNode<DataType>*> ，只存储地址，效率更高且逻辑正确。

#### 2. 引用与右值的冲突

- 错误原因： 报错信息中提到的 conversion to non-const reference type... from rvalue 是因为 C++ 不允许将一个临时对象（右值）绑定到非 const 引用上。
- 具体场景： 在你的 insertHelper(BSTNode<DataType>*& rootPtr, ...) 中， rootPtr 是一个引用。如果你尝试传递一个临时生成的指针或者在某些转换逻辑中产生了临时变量，编译器就会报错。

#### 3. 比较操作符不匹配

- 错误代码： if(node.right != *visited)
- 报错原因： 编译器提示 no match for 'operator!=' 。
  - node.right 是一个指针 ( BSTNode* )。
  - *visited 是一个对象 ( BSTNode )。
  - 编译器不知道如何比较一个“地址”和一个“对象”是否相等，除非你重载了特殊的 operator!= 。
- 解决方法： 统一使用指针比较 node->right != lastVisited 。

#### 4. 递归逻辑导致的死循环（潜在报错/崩溃）

- 错误代码： void preOrder(BSTNode<DataType>* rootPtr = nullptr)
- 逻辑漏洞：
  - 在函数内部，你有这样一行： if (rootPtr == nullptr) rootPtr = this->root; 。
  - 当递归调用 preOrder(rootPtr->left) 时，如果某个节点没有左孩子（即 left 为 nullptr ），进入下一层递归后， rootPtr 又是 nullptr 。
  - 此时逻辑会再次执行 rootPtr = this->root ，导致程序重新从根节点开始遍历，陷入 无限死循环 ，最终导致栈溢出（Stack Overflow）。
- 解决方法： 使用辅助函数 preOrderHelper(BSTNode* node) 处理递归，公有接口 preOrder() 仅负责启动。

#### 5. AVL 旋转逻辑不完整

- 错误原因： 在 turnLL 和 turnRR 中，你修改了局部指针的指向，但没有更新父节点对该位置的引用。
- 解决方法： 利用 BSTNode<DataType>*& node （指针的引用），在旋转完成后执行 node = newNode ，这样可以直接修改父节点中存储的指针地址，确保树的结构真正被改变。

#### 6.非递归遍历优化

![](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202601312348573.png)

逻辑一样，但是改好代码更加简洁

#### 7.模板类编写

**编写模板类是，不支持类的声明与定义分开写**

#### 8. 节点插入失效 (insertHelper)，重要

```
void BSTree<DataType>::insertHelper
(BSTNode<DataType>* rootPtr, const 
DataType& data)
```
- 问题 ： rootPtr 是按值传递的指针。在函数内部执行 rootPtr = new BSTNode<DataType>(data) 仅仅修改了局部副本， 无法真正将新节点挂载到树上 。
- 修复 ：应改为指针的引用 BSTNode<DataType>*& rootPtr 。

#### 9.有关默认参数

C++ 标准明确规定：

- 成员函数的**默认参数表达式不能包含 `this` 指针**
- 因为默认参数是在**编译阶段**就确定的，而 `this` 是运行时才指向具体对象的指针
- 编译器在处理默认参数时，无法知道 `this` 指向哪个对象，所以会报错