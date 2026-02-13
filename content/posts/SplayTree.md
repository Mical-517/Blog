---
title: SplayTree
published: 2026-02-13T16:46:33+08:00
summary: "有利于树的查找策略:张开策略与半张开策略"
cover:
  image: https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202602011340604.png
tags: [树结构]
categories: '数据结构'
draft: false
lang: ''
---

# 关于自适应树

首先树的重要作用在于高效的查找效率，虽然平衡树的一些操作可以帮助我们进行查找，但是当我们考虑要查找信息的频率的时候，就延伸出另一种查找策略：自适应树，即**如果信息A使用频率非常高，但是他处在平衡树较低层次，这是如果我们把树的改变规则变，让查找频率高的信息放在树的上层，这样就可以变相的提高查找信息的频率**，这时候就有两种方法来进行自适应树的构造。

1. 张开策略
2. 半张开策略

- **全展开（Full Splay）**：每次访问一个节点后，通过一系列旋转操作（zig, zig-zig, zig-zag）将该节点移动到树的根位置。
- **半展开（Semi-Splay）**：与全展开类似，但有一个关键区别。在处理 "zig-zig"（即左左或右右）情况时，半展开只执行一次旋转，将访问节点的父节点向上旋转一层。而全展开会执行两次旋转，最终目的是让访问节点上升两层。

## 1.张开”策略(splaying)

该策略根据子节点、父节点和祖父节点之间链接关系的顺序，成对地使用单一旋转。首先，根据被访问节点R、其父节点Q及其祖父节点P（如果有的话）之间的关系，分为三种情况：

情况1：节点R的父节点是根节点。

情况2：同构配置(Homogeneous configuration)。节点R是其父节点Q的左子节点，Q是其父节点P的左子节点，或R和Q都是右子节点。

情况3：异构配置(Heterogeneous configuration)。节点R是其父节点Q的右子节点，Q是其父节点P的左子节点；或R是Q的左子节点，Q是P的右子节点。

该算法以如下方式将被访问节点R移动到树根部：

```c++
splaying(P, Q, R)

while R不是根节点

if R的父节点是根节点

进行单一张开操作，使R围绕其父节点进行旋转(图6-34(a))：

else if R与其前趋同构

进行一次同构张开操作，首先围绕P旋转Q，再围绕Q旋转R(图6-34(b))：

else //如果R与其前趋异构

进行一次异构张开操作，首先围绕Q旋转R，再围绕P旋转R(图6-34(c))：
```

图6-35显示了重新构造树的区别，访问位于图6-33(a)中第5层的节点T。树的形状立即就得到了改进。接着，访问节点R(图6-35(c))，树的形状变得更好了(图6-35(d))。

![myNotes](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202602131700147.png)

“张开”策略关注元素而不是树的形状。在一些元素比其他元素更常用的环境中，该策略执行得很好。如果靠近根部的元素与最底层元素的访问频率差不多，“张开”技术可能并不是最好的选择。在此情况下，强调平衡树的策略比强调元素访问频率的策略更好，此时可以修改“张开”方法。

“半张开(semisplaying)”是“张开”策略的一个修改版本，对于同构的情况，该策略只需要一次旋转，然后继续张开被访问节点的父节点，而不张开节点本身。图6-34(b)显示了该策略。在访问R后，其父节点围绕P进行旋转，之后继续张开节点Q而不是R。此处没有执行R围绕Q的旋转，这与“张开”策略相同。

## 2.半张开策略

核心差异点在于**在zig-zig同侧的处理方法，半张开会旋转当前节点的父节点，然后更新父节点为当前节点继续旋转**,

它揭示了半展开策略的核心。我们以 "左左"（zig-zig）情况为例来解释。

**初始状态**： [gr](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html) (grandparent) -> [par](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html) (parent) -> [node](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html) (你访问的节点)

          gr
         /
        par
       /
      node
**你的代码执行**： [rotateR(node->parent);](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 这个操作是围绕 [par](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 进行右旋。旋转后，[par](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 会成为 [gr](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 的新子节点，而 [node](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 仍然是 [par](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 的子节点。

**旋转后状态**： [par](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 上升了一层，[gr](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 下降了一层。

```
      par
     /   \
  node    gr
```



**关键点来了**： 在这次操作之后，你访问的 [node](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 并没有改变它在树中的父节点（它的父节点仍然是 [par](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html)），它只是随着 [par](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 一起移动了。为了在下一次循环中继续将 [node](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html)（或者说，包含[node](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html)的这个子树）向根节点方向移动，我们需要处理更高层的节点。

此时，[par](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 已经取代了 [gr](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 的位置。所以，我们将 [node](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 指针更新为 [node->parent](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html)（也就是原来的 [par](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html)），这样在下一次 `while` 循环中，程序就会把 [par](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 当作新的目标节点，继续将它向根节点推进。

**总结一下 [node = node->parent](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 的原因**： 在半展开的 zig-zig 操作中，我们旋转的是父节点 [par](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html)。操作完成后，[par](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 上升了一层。为了让下一次循环能继续将整个子树（包括我们最初访问的 [node](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html)）向根推进，我们需要将循环的操作目标更新为已经上升了的 [par](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html)。因此，`node = node->parent` 是为了在下一次迭代中处理更高层的节点。

这与全展开不同。在全展开的 zig-zig 中，会先旋转 [par](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html)，再旋转 [node](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html)，[node](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 会直接上升两层，所以不需要 [node = node->parent](vscode-file://vscode-app/c:/Users/qjy/AppData/Local/Programs/Microsoft VS Code/_/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 这样的赋值。

# 案例分析：文章单词频率系统

![myNotes](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202602131957835.png)

![myNotes](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202602131958422.png)

```c++
/*
    Generalized Splay Tree这是一个自适应的二叉搜索树，主要使用半张开策略来体现自适应树的思想，进而提要查找策略
*/

#pragma once
#include <iostream>
using namespace std;

template<class DataType>
class SplayNode
{
    public:
    SplayNode():left(nullptr),right(nullptr),parent(nullptr){}
    //有参使用const引用形式
    SplayNode(const DataType& key, SplayNode<DataType>* p=nullptr, SplayNode<DataType>* l=nullptr, SplayNode<DataType>* r=nullptr):key(key),parent(p),left(l),right(r){}

    private:
    SplayNode<DataType>* left;
    SplayNode<DataType>* right;
    SplayNode<DataType>* parent;
    DataType key;
};

template<class DataType>
class SplayTree
{
    public:
    SplayTree():root(nullptr){}
    void inorder();    //中序遍历
    void visit(SplayNode<DataType>* node);    //访问函数
    void insert(const DataType& data);    //访问函数
    SplayNode<DataType>* search(const DataType& data);    //查找函数,注意自适应树要进行splay操作
    private:
    SplayNode<DataType>* root;
    void inorder(SplayNode<DataType>* node);    //中序遍历辅助函数
    void semisplay(SplayNode<DataType>* node);  //半张开操作
    void rotateR(SplayNode<DataType>* node);    //右旋操作
    void rotateL(SplayNode<DataType>* node);    //左旋操作
    void continueRotation(SplayNode<DataType>* gr,SplayNode<DataType>* par,SplayNode<DataType>* ch,SplayNode<DataType>* desc);    //继续旋转操作，处理父节点和祖父节点的关系

};

template<class DataType>
void SplayTree<DataType>::inorder()
{
    inorder(this->root);
}

template<class DataType>
void SplayTree<DataType>::inorder(SplayNode<DataType>* node)
{
    if(node==nullptr) return;
    inorder(node->left);
    this->visit(node);
    inorder(node->right);
}

template<class DataType>
void SplayTree<DataType>::visit(SplayNode<DataType>* node)
{
    if(node!=nullptr)
        cout<<node->key<<" ";
}

template<class DataType>
void SplayTree<DataType>::insert(const DataType& data)
{
    SplayNode<DataType> *curr=this->root,*pre=nullptr,*newNode=nullptr;
    while(curr!=nullptr)
    {
        if(curr->key==data) break;
        else if (curr->key>data)
        {
            pre=curr;
            curr=curr->left;
        }
        else
        {
            pre=curr;
            curr=curr->right;
        }
    }
    if(newNode=new SplayNode<DataType>(data,pre))
    {
        if(pre==nullptr) this->root=newNode;
        else if(pre->key>data) pre->left=newNode;
        else pre->right=newNode;
    }
    else
    {
        cerr<<"内存分配失败!"<<endl;
        return;
    }

}

template<class DataType>
SplayNode<DataType>* SplayTree<DataType>::search(const DataType& data)
{
    SplayNode<DataType>* curr=this->root;
    while(curr!=nullptr)
    {
        if(curr->key==data)
        {
            this->semisplay(curr);
            return curr;
        }
        else if(curr->key>data) curr=curr->left;
        else curr=curr->right;
    }
    return nullptr;
}

template<class DataType>
void SplayTree<DataType>::semisplay(SplayNode<DataType>* node)
{
   while(node!=this->root)
   {
        if(node->parent->parent==nullptr) //zig操作：单旋转
        {
            if(node->parent->left==node) //右旋
            {
                rotateR(node);
            }
            else rotateL(node);
        }
        else if(node->parent->left==node)
        {
            if(node->parent->parent->left==node->parent)    //左左情况
            {
                rotateR(node->parent);
                node=node->parent;
            }
            else
            {
                rotateR(node);
                rotateL(node);
            }
        }
        else
        {
            if(node->parent->parent->right==node->parent)   //右右情况
            {
                rotateL(node->parent);
                node=node->parent;
            }
            else
            {
                rotateL(node);
                rotateR(node);
            }
        }
        if(this->root==nullptr) this->root=node;
    }
}

template<class DataType>
void SplayTree<DataType>::rotateR(SplayNode<DataType>* node)
{
    //处理关键逻辑，处理有关parent指向使用一个函数统一处理
    node->parent->left=node->right;
    node->right=node->parent;
    this->continueRotation(node->parent->parent,node->parent,node,node->right->left);
}

template<class DataType>
void SplayTree<DataType>::rotateL(SplayNode<DataType>* node)
{
    //处理旋转节点的逻辑，处理有关parent指向使用一个函数统一处理
    node->parent->right=node->left;
    node->left=node->parent;
    this->continueRotation(node->parent->parent,node->parent,node,node->left->right);
}

template<class DataType>
void SplayTree<DataType>::continueRotation(SplayNode<DataType>* gr,SplayNode<DataType>* par,SplayNode<DataType>* ch,SplayNode<DataType>* desc)
{
    if(gr!=nullptr)
    {
        if(gr->left==ch->parent) gr->left=ch;
        else gr->right=ch; 
    }
    else
    
    {
        this->root=ch;
    }
    if(desc!=nullptr) desc->parent=par;
    par->parent=ch;
    ch->parent=gr;
}
```

```c++
#include <iostream>
#include "genSplay.hpp"
#include <fstream>
#include <cstring>
#include <cstdlib> //eixt()
using namespace std;

class Word
{
public:
    Word() : word(nullptr), freq(1) {}
    bool operator==(const Word &other)
    {
        return strcmp(word, other.word) == 0;
    }
    bool operator>(const Word &other)
    {
        return strcmp(word, other.word) > 0;
    }

private:
    char *word;
    int freq;
    friend class WordSplay;
};

class WordSplay : public SplayTree<Word>
{
public:
    WordSplay() : differentWords(0), wordcnt(0) {}
    void run(ifstream &fIn, char *fileName); // 运行程序
private:
    int differentWords;            // 统计单词个数
    int wordcnt;                   // 统计总单词数
    void visit(SplayNode<Word> *); // 重写visit函数
};

void WordSplay::run(ifstream &fIn, char *fileName)
{
    char s[100];
    char ch = ' ', i;
    Word rec;
    while (!fIn.eof())
    {
        while (true)
        {
            if (!fIn.eof() && !isalpha(ch))
            {
                fIn.get(ch);
            }
            else
            {
                break;
            }
        }
        if (fIn.eof())
        {
            break;
        }
        for (i = 0; !fIn.eof() && isalpha(ch); i++)
        {
            s[i] = toupper(ch);
            fIn.get(ch);
        }
        s[i] = '\0';
        if (!(rec.word = new char[strlen(s) + 1]))
        {
            delete[] rec.word;
            cerr << "NO Room for new Word\n";
            exit(1);
        }
        strcpy_s(rec.word, strlen(s) + 1, s);
        SplayNode<Word> *p = this->search(rec);
        if (p == nullptr)
        {
            this->insert(rec);
        }
        else
            p->value().freq++;
    }
    this->inorder();
    cout << "\n\nFile:" << fileName
         << "\ncontains:" << wordcnt << " word among which "
         << "differentWords" << " are " << differentWords;
}

void WordSplay::visit(SplayNode<Word> *node)
{
    this->differentWords++;
    this->wordcnt += node->value().freq;
}

int main(int argc, char *argv[])
{
    char fileName[80];
    WordSplay splayTree;
    if (argc != 2)
    {
        cout << "Enter the fileName" << endl;
        cin >> fileName;
    }
    else
        strncpy(fileName, argv[1], 80);
    ifstream fIn(fileName);
    if (fIn.fail())
    {
        cerr << "Cannot open " << fileName << endl;
        return 0;
    }
    splayTree.run(fIn, fileName);
    fIn.close();
    return 0;
}

```

# 学到的知识

思考：

1. 首先是数据组织：这是一个自适应树，同过半张开策略构建，**每个节点的信息是一个单词，每个单词有频率以及名字，所以封装成一个类，作为SplayNode的模版参数**

2. 有关虚函数，重新构建虚函数，要保证参数正确
3. 有关文件流函数的知识学习
4. 关于字符数组函数的参数意义的总结（已经写了博客）
5. **理解run函数功能**：可以已有文件里的每个字符，然后进行逻辑判断后加入自适应树