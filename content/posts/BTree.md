---
title: B树家族-BTree
published: 2026-03-08T23:30:18+08:00
summary: "对BTree的理解与简单实现"
cover:
  image: https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202603101422049.png
tags: [BTree]
categories: '数据结构'
draft: false
lang: ''
---



# 多叉树简介

多义树有一个比较有用的版本，其中对所有节点的键值进行排序。m阶的多义查找树也称为n叉查找树，具有以下特性：

(1)每个节点都可以包含m个子节点和m-1个键值。

(2)所有节点中的键值都按升序排列。

(3)前i个子节点中的键值都小于第i个键值。

(4)后m-i个子节点中的键值都大于第i个键值。

记忆点：就是**我们要知道键值与指针是分开存储的**，我们可以想象一组有序的数据，我们从中抽取两个键值取出，这样一组数据是不是就分为三份，抽出的两个键值与三个指针构成一个节点，管理这个三份（每份也是一个节点）。所以m叉树，每个节点至少有m-1个指针，（可以指向空，但是必须有）

关于（3）：**就是我们画出keys[m],pointers[m+1]**,对比就可以发现，p[i]指向的节点全都小于keys[i],这点要好好理解，就是节点的管理是有p管理的，联系keys这是看键值之间的关系



# B树家族

为什么会出现B树

就是因为辅助存储器的读取规则，与二叉树信息的分散，cpu的高速读取信息之间的联系比较差，需要一种数据结构每个节点的信息可充斥辅助存储器一个块，这样有利于数据的查询



## BTeee

### 1.BTree性质

m阶的B树是具有下列特性的多叉查找树：

(1) 除了叶节点之外，根节点至少有两个子树。

(2) 每个非根非叶节点都有k-1个键值和k个指向子树的指针（其中[m/2]≤k≤m）。

(3) 每个叶节点都有k-1个键值（其中[m/2]≤k≤m）。

(4) 所有的叶节点在同一层¹。

这样每个节点至少有一半键值，所以BTree**至少是半满的，层数较少，一定平衡**，层数较少就意味着，辅存跨块访问的次数就少，查询时间就少



### 2.BTree设计

#### 1.节点设计

```c++
template <class KeyType>
class BTreeNode
{
public:
    BTreeNode() = default;

private:
    bool leaf{true};                                  // 记录当前节点是否是叶子节点,因为是自底向上构建，所以是true
    int keyTally{0};                                  // 记录当前节点键值数量
    KeyType keys[M_COUNT - 1];                        // 记录当前节点的键值
    unique_ptr<BTreeNode<KeyType>> pointers[M_COUNT]; // 记录当前节点指向的子节点

    friend class BTree<KeyType>;
};
```



#### 2.B树的查找

**记住p[i]**，指向比当前节点keys[i]小的节点

算法

```c++
BTreenode *BTreeSearch(keyType K, BTreenode *node)
{
    if (node != 0)
    {
        for (i = 1; i <= node->keyTally && node->keys[i - 1] < K; i++)
            ;
        //关键
        if (i > node->keyTally || node->keys[i - 1] > K)
            return BTreeSearch(K, node->pointers[i - 1]);
        else
            return node;
    }
    else
        return 0;
}
```



```c++
// 查询键值
template <class KeyType>
BTreeNode<KeyType> *BTree<KeyType>::searchHelper(const KeyType &data, BTreeNode<KeyType> *currNode, int &index)
{
    // 修复 1：必须判断当前传入的节点 currNode 是否为空，而不是根节点
    if (currNode == nullptr)
    {
        return nullptr;
    }

    int i = 0; // 对应查询键值的下标
               // 在当前节点进行查询
    for (i = 0; i < currNode->keyTally && currNode->keys[i] < data; i++)
    {
        // 空循环
    }
    if (i >= currNode->keyTally || currNode->keys[i] > data)
    {
        return searchHelper(data, currNode->pointers[i].get(), index);
    }
    else
    {
        index = i;
        return currNode;
    }
}

template <class KeyType>
KeyType &BTree<KeyType>::search(const KeyType &data)
{
    int index = 0;
    auto ptr = searchHelper(data, this->node.get(), index);

    if (ptr == nullptr)
    {
        // 修复 2：如果没找到，必须中断执行抛出异常，否则无法返回合法的引用
        throw runtime_error("查找失败：数据不存在");
    }
    else
    {
        return ptr->keys[index];
    }
}
```



#### 3.B树的插入（重要复杂）

因为所有的叶节点都必须位于B树的最后一层，所以插入和删除操作并不简单。改变创建树的策略有利于简化插入操作的实现。向二叉查找树中插入节点时，树往往是自顶向下建立的，但会得到不平衡的树。如果树的第一个键值是最小的一个，则该键值会放在根节点中，而且该树没有左子树，除非为了平衡树采取特殊的防范措施。

**树还可以自底向上建立**，于是根将处于不断变化之中，只有当所有的插入完成后，才能确定根的内容。在B树中插入键值也会应用这种策略，在此过程时，对于要插入的键值，直接将其放到尚有空间的叶节点中，如果叶节点已满，就创建另一个叶节点，键值在这些叶节点中重新分配，并将一个键值提升到父节点中。如果父节点已满，则重复刚才的过程，直到到达根节点，并创建一个新的根节点。

插入的三种情况：

![image-20260310144859977](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202603101449072.png)

**对应的算法**

```c++
BTreInsert(K)
{
    // 找到一个叶节点node来插入K;
    while (true)
        // 在数组keys中为K找到一个合适的位置；
        if node不满
            // 插入K并递增keyTally;
            return;
        else
            // 将node分解为node1和node2;
            //  node1=node, node2是新节点；
            // 在node1和node2之间平均分配键值和指针，并正确地初始化其keyTally;
            K = 中间键值；
            if node 是根节点
                //创建一个新的根节点，作为node1和node2的父节点；
                //将K及指向node1和node2的指针放在根节点中，并将根节点的keyTally设置为1;
    		return;
    	    else node = 其父节点； // 现在开始处理node的父节点；
}
```

**算法实现**

##### 1.先找到叶节点进行插入

首先：

1. 插入是自底向上构建，但是每个节点没有存储对应的父节点指针，所以在查找的过程要维护这一个搜索路径，以便后续向上构建
2. b树不允许重复键值，所以在找到过程要有限制条件

```c++
// 插入查询,要适配向上构建，所以但是插入是非递归的，所以要记录从上到下的路径
template <class KeyType>
BTreeNode<KeyType> *BTree<KeyType>::insertSearch(const KeyType &data, vector<BTreeNode<KeyType> *> &path)
{
    BTreeNode<KeyType> *currNode = this->node.get();
    if (currNode == nullptr)
        return nullptr;
    while (true)
    {
        path.push_back(currNode);
        // 先找到第一个大于等于data的键值的下标
        int i = 0;
        while (i < currNode->keyTally && data > currNode->keys[i])
            i++;

        // 判断如果键值重复
        if (currNode->keys[i] == data)
        {
            cout << "Key" << data << "already exists." << endl;
            return nullptr;
        }
        /*
        //如果还有子节点就往下探索
        if(currNode->pointers[i]!=nullptr)
        {
            insertSearch(data,path);
        }
        */
        // 如果是叶子结点，就表明找到了(关键条件)
        if (currNode->leaf)
        {
            return currNode;
        }
        // 不是就继续向下探索
        else
        {
            currNode = currNode->pointers[i].get();
        }
    }
}
```

##### 2.插入的第一种情况

如果插入的节点keys数组有空，插入之后要维护的是

1. 保证keys有序
2. 维护原有的指向下层节点的指针
3. 可能这个插入的键值是从下面节点分裂出来的，所以还要管理下层节点新new出来的节点

```c++
/ 定义变量记录路径，以便后续向上构建
    vector<BTreeNode<KeyType> *> path;
    // 找到要插入的节点
    BTreeNode<KeyType> *targetNode = insertSearch(data, path);

    // 如果为空就表明，键值重复
    if (targetNode == nullptr)
        return;

    // 开始插入，定义变量表示可能产生的新节点
    unique_ptr<BTreeNode<KeyType>> newChildNode = nullptr; // 分裂产生的新子节点
    KeyType keyToInsert = data;

    while (true)
    {
        // 如果当前节点的键值数组还有空
        if (targetNode->keyTally < M_COUNT - 1)
        {
            // 找到插入位置然后重新排序
            int i = targetNode->keyTally;
            while (i > 0 && keyToInsert < targetNode->keys[i - 1])
            {
                targetNode->keys[i] = targetNode->keys[i - 1];
                // 如果他不是叶子节点的话，要把ptr[i]向后移动
                if (!targetNode->leaf)
                {
                    targetNode->pointers[i + 1] = move(targetNode->pointers[i]);
                }
                i--;
            }
            targetNode->keys[i] = keyToInsert;
            // 如果此时有新子节点,当前节点的智能指针就要管理他,并且更新是否是叶子节点
            if (newChildNode != nullptr)
            {
                targetNode->pointers[i + 1] = move(newChildNode);
                targetNode->leaf = false;
            }
            targetNode->keyTally++;
            return;
        }
```

关键代码理解

```c++
                targetNode->keys[i] = targetNode->keys[i - 1];
                // 如果他不是叶子节点的话，要把ptr[i]向后移动
                if (!targetNode->leaf)
                {
                    targetNode->pointers[i + 1] = move(targetNode->pointers[i]);
                }
                i--;


当我们插入的keys[i]比keys[i-1]小，所以keys[i-1]后移，同时，原先的ptr[i]指向的比key[i-1]大的节点，所以他一定比keys[i]大，所以ptr[i]同步向后移动


___________________________________________________________________________________________
            // 如果此时有新子节点,当前节点的智能指针就要管理他,并且更新是否是叶子节点
            if (newChildNode != nullptr)
            {
                targetNode->pointers[i + 1] = move(newChildNode);
                targetNode->leaf = false;
            }
此时键值插入的位置是下标i
如果此时有新的节点，就表示下面某一个节点分成两份，一份比插入的键值小，一份比插入的键值大，第一份已经有指针管理所以不用管，但是第二份是比插入的键值大的，还有一个关键就是这份新的节点归根节点还是原先下层的节点，所以他还是比keys[i+1]小的，所以ptr[i+1]要将接管这份内存管理

```



##### 3.插入的第二种情况

当前节点已经满了，所以当前节点要分裂成两个，然后找到中间键值，插入到父节点，需要维护

1. 分裂两个数组有序性
2. 关于原先指向下层节点指针的分配
3. 如果下层也是满的，也有一个新数组，指针的分配

```c++
else
        {
            // 定义一个新的临时存储所有键值的keys和pointers;
            vector<KeyType> tempKeys;
            vector<unique_ptr<BTreeNode<KeyType>>> tempPointers;

            // 将当前节点所有的键值与指针转移
            for (int i = 0; i < targetNode->keyTally; i++)
            {
                tempKeys.push_back(targetNode->keys[i]);
            }
            for (int i = 0; i <= targetNode->keyTally; i++) // 注意这里是 <=，循环次数是 tally + 1
            {
                tempPointers.push_back(move(targetNode->pointers[i]));
            }

            // 找到新节点的插入位置并更新指针
            auto it_key = lower_bound(tempKeys.begin(), tempKeys.end(), keyToInsert);
            int insert_pos = distance(tempKeys.begin(), it_key); // 修正：tempKeys.begin()

            // 插入键值以及带来的新的子节点
            tempKeys.insert(tempKeys.begin() + insert_pos, keyToInsert);
            tempPointers.insert(tempPointers.begin() + insert_pos + 1, move(newChildNode));

            // 定义分裂点,确定要提升到父节点的键值
            int splitIndex = M_COUNT / 2;
            KeyType keyToParent = tempKeys.at(splitIndex);

            // 创建新的右侧兄弟节点
            auto rightNode = make_unique<BTreeNode<KeyType>>();
            rightNode->leaf = targetNode->leaf;

            // 填充旧节点（作为左节点）
            targetNode->keyTally = splitIndex;
            for (int i = 0; i < splitIndex; i++) // 修正：大小写统一
            {
                targetNode->keys[i] = tempKeys[i];
                targetNode->pointers[i] = move(tempPointers[i]);
            }
            targetNode->pointers[splitIndex] = move(tempPointers[splitIndex]);

            // 填充新的右节点 (修正核心计算逻辑：减去提升的1个键)
            rightNode->keyTally = M_COUNT - 1 - splitIndex;
            for (int i = 0; i < rightNode->keyTally; i++) // 修正：分号
            {
                rightNode->keys[i] = tempKeys[splitIndex + 1 + i];
                rightNode->pointers[i] = move(tempPointers[splitIndex + 1 + i]);
            }
            rightNode->pointers[rightNode->keyTally] = move(tempPointers[M_COUNT]);

            // 准备向上传递
            keyToInsert = keyToParent;
            newChildNode = move(rightNode);

            // 从路径中移除当前节点
            path.pop_back(); // 修正：vector用pop_back()
```



关键代码解读：

```c++
            // 找到新节点的插入位置并更新指针
            auto it_key = lower_bound(tempKeys.begin(), tempKeys.end(), keyToInsert);
            int insert_pos = distance(tempKeys.begin(), it_key); // 修正：tempKeys.begin()

            // 插入键值以及带来的新的子节点
            tempKeys.insert(tempKeys.begin() + insert_pos, keyToInsert);
            tempPointers.insert(tempPointers.begin() + insert_pos + 1, move(newChildNode));
-------------------------------------------------------
 假如键值插入位置是i，那么就代表他比i+1位置上键值小，这里就会有一个疑问（关键**********）
    下面传来的新节点是比插入的键值大，这是肯定的，但是他一定比i+1上的键值小吗？
    这是一定的********
    首先记住一点，新节点就是原本从下层节点分裂出来的，好那么，假设管理这个下层节点的下标为j，那么原理当前节点的keys[j]比这个下层节点所有键值都要大对吧，keys[j-1]比这个下层节点所有键值都要小，好现在从下层节点出来一个键值插入到当前节点的i位置，他插入的位置是不是就是比j-1多一位的位置，就是下标就是j，所以这里i就是j，所以原来的j后移变成j+1，就是i+1一定比新节点所有值都大
```

##### 4.插入的第三种情况

当前节点是根节点，并且满了，所以此时就要new两个节点，一个是分裂，一个新的根节点

```c++
// 如果当前节点是根节点 (通过路径是否为空来判断最稳妥)
            if (path.empty())
            {
                auto newRoot = make_unique<BTreeNode<KeyType>>(); // 修正：KeyType
                newRoot->keyTally = 1;
                newRoot->leaf = false;
                newRoot->keys[0] = keyToInsert; // 修正：提取上来的应当直接赋值
                newRoot->pointers[0] = move(this->node);
                newRoot->pointers[1] = move(newChildNode); // 修正：pointers拼写
                this->node = move(newRoot);
                return;
            }
            else
            {
                // 处理targetNode的父节点
                targetNode = path.back();
            }
```





