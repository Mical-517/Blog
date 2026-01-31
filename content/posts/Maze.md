---
title: Maze
published: 2026-01-31T19:38:37+08:00
summary: "迷宫系统"
cover:
  image: https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202601300127015.png
tags: [stack]
categories: '数据结构'
draft: false
lang: ''
---

```c++
#pragma once
#include <iostream>
#include <stack>
#include <cstring>
using namespace std;

class Maze;

//迷宫中每个方块类
class Block
{
    public:
    Block() : x(0), y(0) {}
    Block(int x,int y)
    {
        this->x=x;
        this->y=y;
    }

    bool operator==(const Block& block) const
    {
        return this->x==block.x&&this->y==block.y;
    }
    private:
    //方块的位置信息
    int x = 0, y = 0;
    //当前方块可以探索的方向，用数组表示,0~3表示上下左右，存储0表示为探索，1表示探索过
    int direction[4]{0,0,0,0};

    friend class Maze;
};

//迷宫类
class Maze
{
    public:
    //构造函数,根据用户输入的迷宫大小自动初始化
    Maze();
    //出口函数
    void exitMaze();

    private:
    //迷宫中的入口，出口，当前位置信息
    Block entry,exit,current;
    //指定迷宫的标志符号
    const char exitMark,entryMark,wallMark,pathMark,visitedMark;
    //迷宫二维数组
    char** maze;
    int row=0,col=0;
    //迷宫中保存路径的栈
    stack<Block> path;
    //方向常量,顺时针
    const int dx[4]={-1,0,1,0};
    const int dy[4]={0,1,0,-1};

    //判断当前位置是否有可走方向
    bool hasNextDirection(Block& currentBlock,Block& nextBlock);
    //判断方块是否可通行
    bool isPassable(int i,int j);
    //重置当前方块的探索方向
    void resetDirection(Block& currentBlock);
    //打印路径
    void printPath();
};

//构造函数
Maze::Maze():exitMark('e'),entryMark('m'),visitedMark('.'),wallMark('0'),pathMark('1')
{
    //提示信息：
    cout<<"now input the maze and rules follow the format:"<<endl;
    cout<<"exitMark:e"<<endl;
    cout<<"entryMark:m"<<endl;
    cout<<"wallMark:0"<<endl;
    cout<<"pathMark:1"<<endl;
    //临时变量，存储用户输入的行
    char str[100];
    char* s;
    int rows=0,cols=0;
    stack<char*> mazeRows;  //用来存放最终迷宫的每一行的指向

    while(cin>>str && strcmp(str, "stop") != 0) // 增加停止条件
    {
        rows++;
        int currentCols = strlen(str);
        if (currentCols > cols) cols = currentCols; // 记录最大列数
        
        s=new char[currentCols+3];   // 在两端加墙
        memcpy(s + 1, str, currentCols);
        s[0] = s[currentCols+1] = this->wallMark;
        s[currentCols+2] = '\0'; 

        mazeRows.push(s);
        
        char* exitPos=strchr(s,exitMark);
        char* entryPos=strchr(s,entryMark);
        if(exitPos!=nullptr)
        {
            this->exit=Block(rows,exitPos-s);
        }
        if(entryPos!=nullptr)
        {
            this->entry=Block(rows,entryPos-s);
        }
    }
    //将stack中字符数组复制到maze中
    this->row=rows+2;
    this->col=cols+2; 
    this->maze=new char*[this->row];
    
    // 关键修复：从下往上填充中间行，索引 1 到 rows
    for(int i = rows; i >= 1; i--)
    {
        this->maze[i]=mazeRows.top();
        mazeRows.pop();
    }

    // 分配并填充顶层和底层围墙
    this->maze[0] = new char[this->col + 1];
    this->maze[this->row - 1] = new char[this->col + 1];
    for(int j = 0; j < this->col; j++)
    {
        this->maze[0][j] = wallMark;
        this->maze[this->row - 1][j] = wallMark;
    }
    this->maze[0][this->col] = this->maze[this->row - 1][this->col] = '\0';
}

void Maze::exitMaze()
{
    // 标记入口并压栈
    this->maze[this->entry.x][this->entry.y] = visitedMark;
    this->current = this->entry;
    this->path.push(this->current);

    while (!this->path.empty())
    {
        // 1. 判断是否到达出口
        if (current == exit)
        {
            printPath();
            // 为了寻找所有路径，到达出口后需要“假装”没路了，触发回溯
            // 回溯前将当前点标记回出口标志，以便其他路径也能认出它
            this->maze[current.x][current.y] = exitMark;
            this->path.pop();
            if (!this->path.empty())
            {
                current = this->path.top();
            }
            continue;
        }

        // 2. 尝试向下一个方向移动
        Block next;
        if (hasNextDirection(current, next))
        {
            // 在入栈前，更新当前块在栈中的状态（方向已经更新过了）
            this->path.pop();
            this->path.push(current);

            // 移动到新块
            current = next;
            // 只有普通路径点才标记为已访问，避免覆盖出口/入口标志
            if (this->maze[current.x][current.y] == pathMark)
            {
                this->maze[current.x][current.y] = visitedMark;
            }
            this->path.push(current);
        }
        else
        {
            // 3. 无路可走，回溯
            // 回溯时撤销当前块的“已访问”标记，使其能被其他路径再次使用
            if (this->maze[current.x][current.y] == visitedMark)
            {
                this->maze[current.x][current.y] = pathMark;
            }
            resetDirection(current);
            this->path.pop();
            if (!this->path.empty())
            {
                current = this->path.top();
            }
        }
    }
}

bool Maze::hasNextDirection(Block& currentBlock,Block& nextBlock)
{
    for(int i=0;i<4;i++)
    {
        if (currentBlock.direction[i] == 0) // 必须检查该方向是否已尝试过
        {
            currentBlock.direction[i]=1;
            int nx = currentBlock.x + dx[i];
            int ny = currentBlock.y + dy[i];
            if(isPassable(nx, ny))
            {
                nextBlock.x = nx;
                nextBlock.y = ny;
                return true;
            }
        }
    }
    return false;
}

bool Maze::isPassable(int i,int j)
{
    if(this->maze[i][j]==wallMark||this->maze[i][j]==visitedMark)
    {
        return false;
    }
    return true;
}

void Maze::resetDirection(Block& currentBlock)
{
    for(int i=0;i<4;i++)
    {
        currentBlock.direction[i]=0;
    }
    //重置访问标记
    this->maze[currentBlock.x][currentBlock.y]=pathMark;
}

void Maze::printPath()
{
    // 栈中保存了从起点到当前点的所有坐标
    stack<Block> tempPath(this->path);
    stack<Block> reversePath;
    while (!tempPath.empty())
    {
        reversePath.push(tempPath.top());
        tempPath.pop();
    }

    cout << "Path found: ";
    while (!reversePath.empty())
    {
        cout << "(" << reversePath.top().x << "," << reversePath.top().y << ")";
        reversePath.pop();
        if (!reversePath.empty()) cout << " -> ";
    }
    cout << endl;
}
```





### 一， 设计思路：

**基于回溯的迷宫多路径的搜索算法，核心是将迷宫抽象成方块点的集合，利用栈来模拟搜索过程**

1. 节点状态化 (Block 类) ：
   - 不仅存储坐标 (x, y) ，还将“探索进度”集成在 direction[4] 数组中。这使得每个节点都具备了“记忆”能力，知道哪些方向已经尝试过，是实现非递归回溯的关键。
2. 迷宫自动化构建 (Maze 构造函数) ：
   - 设计了一个动态增长的迷宫构建逻辑，支持从控制台读取任意大小的迷宫。
   - 防御性设计 ：通过在用户输入的迷宫四周额外包裹一层“墙壁”（Wall），简化了边界判定逻辑，使得搜索时无需频繁检查数组下标是否越界。
3. 栈驱动的非递归搜索 (exitMaze 函数) ：
   - 使用 std::stack 保存当前路径。
   - 移动策略 ：深度优先探索（DFS），只要当前点有可走方向就继续前进。
   - 回溯策略 ：当某点无路可走或到达出口时，通过弹栈（Pop）退回到上一个节点，利用节点自带的方向记忆继续尝试其他分支。

### 二、 修改过程中的关键问题复盘

在调试和优化过程中，我们主要解决了以下四个维度的缺陷：
 1. 对象的生命周期与初始化 (C++ 核心语法)
- 问题 ： Block 类缺少默认构造函数，导致 Maze 类在构造时无法初始化其成员。
- 教训 ：在 C++ 中，如果你定义了带参构造函数，编译器就不会再自动生成默认构造函数。如果该类被作为其他类的成员变量，必须确保它有默认构造函数，或者在初始化列表中显式初始化。 2. 内存布局与字符串偏移 (底层细节)
- 问题 ：在构造函数中， strcpy 和坐标计算 exitPos - s + 1 导致了数据覆盖和索引错位。
- 分析 ：你在左右加墙时，没有给原始字符串留出偏移量（Offset），直接覆盖了 s[0] 。
- 修正 ：改用 memcpy(s + 1, str, len) 将原始数据精准放置在带墙数组的中间，确保坐标系 (row, col) 与实际内存索引完全对齐。 3. 搜索算法的死循环 (逻辑严谨性)
- 问题 ：程序在两个相邻空地之间无限横跳。
- 根源 ：
  1. 状态检查缺失 ：每次进入方块都从第一个方向开始看，没有检查 direction[i] == 0 。
  2. 访问标记同步 ：移动到新方块时没有及时在 maze 数组中打上 visited 标记。
- 修正 ：在 hasNextDirection 中严格检查方向数组，并在 exitMaze 移动时同步更新迷宫状态。 4. 全路径搜索与标记撤销 (算法深度)
- 问题 ：只能找到第一条路径，之后就再也找不到了。
- 根源 ：找到路径后，路径上的方块被永久标记为“已访问”。
- 修正 ：引入了**回溯撤销（Unmark）**机制。在 exitMaze 弹栈回溯时，将 visitedMark 还原为 pathMark 。
- 意义 ：这让方块能够被其他分支路径重新利用，从而实现了从“寻路”到“遍历所有路径”的质变。
### 三、 总结

你的初始设计非常大胆且具有工程感（特别是自动包裹围墙的设计）。代码的主要挑战在于 非递归状态机 的维护，这比递归写法更难排查状态同步问题。

通过这几次修改，代码从一个“容易崩溃且只能找单路径”的草稿，变成了一个能够 自动对齐、防御越界、稳定搜索全路径 的健壮程序。建议在后续开发中，多关注 栈顶状态与物理数组状态的强一致性 ，这是处理这类问题的核心。

**关键在于自己要明白入栈是在当前块是否被探索之前还是在被探索之后**