---
title: Graph
published: 2026-02-14T11:15:16+08:00
summary: "有关于图的知识"
cover:
  image: https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202601300127015.png
tags: [Graph]
categories: '数据结构'
draft: false
lang: ''
---

# 图的简介

尽管树很灵活并且有许多不同的应用，但是树本身存在局限，树只能表示层次关系，例如父子关系。其他关系只能间接表示，例如同级关系。图是树的推广，在这种数据结构中不存在这一限制。直观地讲，图是由顶点(或节点)及顶点间的关系组成的集合。

简单图G=(V,E)由非空顶点集V和可空边(edge)集E组成。每条边都是V中两个顶点的集合。顶点和边的数量分别用|V|和|E|表示。有向图G=(V,E)由非空顶点集和可空边集E组成(有向图的边也称为弧)，每条边包含V中的一对顶点。不同的是，简单图边的形式为{v，v}，且{v，v}={v，v}。在有向图中，边的形式为(v，v)，且(v，v)≠(v，v)。若非必要，记号的区别可以忽略，顶点vi和vj之间的边可以表示为edge(vi，vj)。

如果图的每条边都附带有数值，则称为加权图(weighted graph)。根据使用这种图的具体环境，边所附带的数值可以称为边的权、成本、顶点间的距离、长度等。

# 图的表示方法

## 1.通用表示方法

### 1.邻接矩阵

$$
\
a_{ij}=
\begin{cases}
1 & \text{若存在边 } \text{edge}(v_i, v_j) \\
0 & \text{其他情况}
\end{cases}
$$

这是最直观的存储方式，核心是用二维数组（矩阵）表示顶点间的连接关系。

- **原理**：

  - 设图有 `n` 个顶点，定义二维数组 `matrix[n][n]`。
  - 对于**有向图**：`matrix[i][j] = 1`（或边的权重）表示存在从顶点 `i` 指向 `j` 的边；`matrix[i][j] = 0` 表示无此边。
  - 对于**无向图**：因为边是双向的，所以 `matrix[i][j] = matrix[j][i]`（有边则均为 1 / 权重，无边则均为 0）。

  

- **适用场景**：顶点数量少、边多的**稠密图**（边数接近 `n²`）。



代码实现

```c++
#include <iostream>
#include <vector>
using namespace std;

// 邻接矩阵存储图
class GraphMatrix {
private:
    int n; // 顶点数量
    vector<vector<int>> matrix; // 邻接矩阵，0表示无边，非0表示权重
    bool isDirected; // 是否为有向图

public:
    // 构造函数：初始化n个顶点的空图
    GraphMatrix(int vertexNum, bool directed = false) : n(vertexNum), isDirected(directed) {
        // 初始化矩阵为全0（无边）
        matrix.resize(n, vector<int>(n, 0));
    }

    // 添加边：u->v（有向）或u-v（无向），weight为边的权重
    void addEdge(int u, int v, int weight = 1) {
        // 检查顶点合法性
        if (u < 0 || u >= n || v < 0 || v >= n) {
            cout << "顶点编号非法！" << endl;
            return;
        }
        matrix[u][v] = weight;
        // 无向图需要双向赋值
        if (!isDirected) {
            matrix[v][u] = weight;
        }
    }

    // 打印邻接矩阵
    void print() {
        cout << "邻接矩阵：" << endl;
        for (int i = 0; i < n; ++i) {
            for (int j = 0; j < n; ++j) {
                cout << matrix[i][j] << " ";
            }
            cout << endl;
        }
    }
};

// 测试
int main() {
    // 1. 测试无向图
    cout << "=== 无向图示例 ===" << endl;
    GraphMatrix undirGraph(4, false); // 4个顶点的无向图
    undirGraph.addEdge(0, 1); // 边0-1
    undirGraph.addEdge(0, 2, 5); // 边0-2，权重5
    undirGraph.addEdge(1, 3); // 边1-3
    undirGraph.print();

    // 2. 测试有向图
    cout << "\n=== 有向图示例 ===" << endl;
    GraphMatrix dirGraph(3, true); // 3个顶点的有向图
    dirGraph.addEdge(0, 1); // 边0→1
    dirGraph.addEdge(1, 2, 3); // 边1→2，权重3
    dirGraph.print();

    return 0;
}
```





### 2.邻接表

![myNotes](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202602141128101.png)

这是工程中最常用的存储方式，核心是「数组 + 链表 / 向量」，空间效率远高于邻接矩阵。

- **原理**：

  - 定义数组（或 vector）`adj`，`adj[i]` 存储顶点 `i` 的所有邻接顶点（及权重）。
  - 对于**有向图**：`adj[i]` 只存储 `i` 指向的顶点。
  - 对于**无向图**：添加边 `i-j` 时，需在 `adj[i]` 中加 `j`，同时在 `adj[j]` 中加 `i`。

  

- **适用场景**：顶点数量多、边少的**稀疏图**（边数远小于 `n²`），如社交网络、路网等。

```c++
#include <iostream>
#include <vector>
#include <utility> // 用于pair
using namespace std;

// 邻接表存储图
class GraphList {
private:
    int n; // 顶点数量
    // adj[i] 存储 (邻接顶点, 边权重) 的pair
    vector<vector<pair<int, int>>> adj;
    bool isDirected; // 是否为有向图

public:
    GraphList(int vertexNum, bool directed = false) : n(vertexNum), isDirected(directed) {
        adj.resize(n); // 初始化n个空的邻接表
    }

    // 添加边：u->v（有向）或u-v（无向）
    void addEdge(int u, int v, int weight = 1) {
        if (u < 0 || u >= n || v < 0 || v >= n) {
            cout << "顶点编号非法！" << endl;
            return;
        }
        adj[u].emplace_back(v, weight); // 存储u的邻接顶点v和权重
        // 无向图需要反向添加
        if (!isDirected) {
            adj[v].emplace_back(u, weight);
        }
    }

    // 打印邻接表
    void print() {
        cout << "邻接表：" << endl;
        for (int i = 0; i < n; ++i) {
            cout << "顶点 " << i << " -> ";
            for (auto& p : adj[i]) {
                cout << "(" << p.first << ", 权重" << p.second << ") ";
            }
            cout << endl;
        }
    }
};

// 测试
int main() {
    // 1. 无向图
    cout << "=== 无向图示例 ===" << endl;
    GraphList undirGraph(4, false);
    undirGraph.addEdge(0, 1);
    undirGraph.addEdge(0, 2, 5);
    undirGraph.addEdge(1, 3);
    undirGraph.print();

    // 2. 有向图
    cout << "\n=== 有向图示例 ===" << endl;
    GraphList dirGraph(3, true);
    dirGraph.addEdge(0, 1);
    dirGraph.addEdge(1, 2, 3);
    dirGraph.print();

    return 0;
}
```

## 2.针对有向图的十字链表

优势：

1. 有向图的普通邻接表只能快速遍历一个顶点的**出边**（比如顶点`i`指向哪些顶点），但要找**入边**（哪些顶点指向`i`），需要遍历整个邻接表，效率很低。

   十字链表的核心是：**把每条有向边的 “出边属性” 和 “入边属性” 整合到一个节点中**，同时给每个顶点维护 “出边表头指针” 和 “入边表头指针”，实现 “一次存储，双向快速查询”。

2. 需要**同时高效处理有向图的出边和入边**（比如拓扑排序、有向图的强连通分量分析、编译原理的语法分析）

成员结构：

对于有向图来说，邻接表（逆邻接表）是有缺陷的，其仅仅表述了关于出度（入度）的信息，如果想要了解另一方面就必须遍历整个图才可以得知。因此十字链表（Orthogonal List）应运而生，可以看做是邻接表和逆邻接表的结合。

1. 边节点（ArcNode）

存储一条有向边的信息，包含 4 个核心字段：

- `vex1`：边的起点（尾顶点）
- `vex2`：边的终点（头顶点）
- `outlink`：指针，指向 “同起点” 的下一条边（出边链）
- `inlink`：指针，指向 “同终点” 的下一条边（入边链）

2. 顶点节点（VexNode）

存储顶点信息，包含 3 个核心字段：

- `data`：顶点的数据（比如编号 / 名称）
- `firstout`：指针，指向该顶点的第一条出边（出边链的头）
- `firstin`：指针，指向该顶点的第一条入边（入边链的头）

```c++
template <class DataType, class WeightType>
class DiGraph;

// 边集节点
template <class WeightType>
class EdgeNode
{
public:
    EdgeNode(int headIndex = 0, int tailIndex = 0,
             EdgeNode<WeightType> *headEdge = nullptr,
             EdgeNode<WeightType> *tailEdge = nullptr,
             WeightType weight = WeightType());
    EdgeNode() = default;

private:
    // 让所有 DiGraph 模板实例成为友元（避免特化在类内引起编译器错误）
    template <class D, class W>
    friend class DiGraph;

    int headIndex;
    int tailIndex;
    EdgeNode<WeightType> *headEdge;
    EdgeNode<WeightType> *tailEdge;
    WeightType weight;
};

// 顶点节点
template <class DataType, class WeightType>
class VertexNode
{
public:
    VertexNode(DataType data = DataType(),
               EdgeNode<WeightType> *firstIn = nullptr,
               EdgeNode<WeightType> *firstOut = nullptr);

private:
    DataType data;
    EdgeNode<WeightType> *firstIn;
    EdgeNode<WeightType> *firstOut;
    friend class DiGraph<DataType, WeightType>;
};

// 十字链表有向图
template <class DataType, class WeightType>
class DiGraph
{
public:
    DiGraph(int vertexCapacity = 0);
    ~DiGraph()
    {
        this->clear();
        if (vertexArry)
            delete[] vertexArry;
    }

    // 添加顶点
    void addVertex(const DataType data);
    // 添加边
    void addEdge(const DataType head, const DataType tail, const WeightType weight);
    // 删除边
    void removeEdge(const DataType head, const DataType tail);
    // 清空函数
    void clear();
    // 打印函数
    void print();
    // 出度与入度函数
    int outDegree(const DataType data);
    int inDegree(const DataType data);

private:
    VertexNode<DataType, WeightType> *vertexArry = nullptr;
    int vertexNum = 0;
    int capacity = 0;
    // 利用顶点存储信息查询对应节点下标
    int search(const DataType data);
};

// 边节点构造函数
template <class WeightType>
EdgeNode<WeightType>::EdgeNode(int headIndex, int tailIndex,
                               EdgeNode<WeightType> *headEdge,
                               EdgeNode<WeightType> *tailEdge,
                               WeightType weight)
    : headIndex(headIndex), tailIndex(tailIndex),
      headEdge(headEdge), tailEdge(tailEdge), weight(weight)
{
}

// 顶点节点构造函数
template <class DataType, class WeightType>
VertexNode<DataType, WeightType>::VertexNode(DataType data,
                                             EdgeNode<WeightType> *firstIn,
                                             EdgeNode<WeightType> *firstOut)
    : data(data), firstIn(firstIn), firstOut(firstOut)
{
}

// 十字链表有向图构造函数
template <class DataType, class WeightType>
DiGraph<DataType, WeightType>::DiGraph(int vertexCapacity)
{
    this->vertexNum = 0;
    this->capacity = vertexCapacity > 0 ? vertexCapacity : 0;
    if (this->capacity > 0)
        vertexArry = new VertexNode<DataType, WeightType>[this->capacity];
    else
        vertexArry = nullptr;

    // (new 会在失败时抛出异常，保留检查以防编译器行为不同)
    if (this->capacity > 0 && vertexArry == nullptr)
    {
        cerr << "Memory allocation failed\n";
        exit(1);
    }
}

// 添加边
template <class DataType, class WeightType>
void DiGraph<DataType, WeightType>::addEdge(const DataType head, const DataType tail, const WeightType weight)
{
    int headIndex = search(head);
    int tailIndex = search(tail);
    if (headIndex == -1 || tailIndex == -1)
    {
        cerr << "Vertex Not Found\n";
        return;
    }
    EdgeNode<WeightType> *newEdge =
        new EdgeNode<WeightType>(headIndex, tailIndex,
                                 this->vertexArry[headIndex].firstOut,
                                 this->vertexArry[tailIndex].firstIn,
                                 weight);
    if (newEdge == nullptr)
    {
        cerr << "Memory allocation failed\n";
        exit(1);
    }
    this->vertexArry[headIndex].firstOut = newEdge;
    this->vertexArry[tailIndex].firstIn = newEdge;
}

// search函数
template <class DataType, class WeightType>
int DiGraph<DataType, WeightType>::search(const DataType data)
{
    for (int i = 0; i < this->vertexNum; i++)
    {
        if (vertexArry[i].data == data)
            return i;
    }
    return -1;
}

// 清空函数
template <class DataType, class WeightType>
void DiGraph<DataType, WeightType>::clear()
{
    if (!vertexArry)
        return;
    EdgeNode<WeightType> *curr = nullptr, *pre = nullptr;
    for (int i = 0; i < this->vertexNum; i++)
    {
        curr = vertexArry[i].firstOut;
        while (curr != nullptr)
        {
            pre = curr;
            curr = curr->headEdge;
            delete pre;
        }
        vertexArry[i].firstOut = nullptr;
        vertexArry[i].firstIn = nullptr;
    }
    vertexNum = 0;
}

// 打印函数
template <class DataType, class WeightType>
void DiGraph<DataType, WeightType>::print()
{
    if (!vertexArry)
    {
        cout << "(empty graph)\n";
        return;
    }
    EdgeNode<WeightType> *curr = nullptr;
    for (int i = 0; i < this->vertexNum; i++)
    {
        cout << vertexArry[i].data << ": ";
        curr = vertexArry[i].firstOut;
        while (curr != nullptr)
        {
            cout << vertexArry[curr->tailIndex].data << "(" << curr->weight << ") ";
            curr = curr->headEdge;
        }
        cout << " OutDegree:" << outDegree(vertexArry[i].data)
             << " InDegree:" << inDegree(vertexArry[i].data) << "\n";
    }
}

// 出入度函数
template <class DataType, class WeightType>
int DiGraph<DataType, WeightType>::outDegree(const DataType data)
{
    int index = search(data);
    if (index == -1)
    {
        cerr << "Vertex Not Found\n";
        return -1;
    }
    int degree = 0;
    EdgeNode<WeightType> *curr = vertexArry[index].firstOut;
    while (curr != nullptr)
    {
        degree++;
        curr = curr->headEdge;
    }
    return degree;
}

template <class DataType, class WeightType>
int DiGraph<DataType, WeightType>::inDegree(const DataType data)
{
    int index = search(data);
    if (index == -1)
    {
        cerr << "Vertex Not Found\n";
        return -1;
    }
    int degree = 0;
    EdgeNode<WeightType> *curr = vertexArry[index].firstIn;
    while (curr != nullptr)
    {
        degree++;
        curr = curr->tailEdge;
    }
    return degree;
}

// 添加顶点函数
template <class DataType, class WeightType>
void DiGraph<DataType, WeightType>::addVertex(const DataType data)
{
    if (this->capacity == 0 || this->vertexNum == this->capacity)
    {
        cerr << "Vertex Capacity Exceeded\n";
        return;
    }
    if (search(data) != -1)
    {
        cerr << "Vertex Already Exists\n";
        return;
    }
    vertexArry[this->vertexNum].data = data;
    vertexArry[this->vertexNum].firstIn = nullptr;
    vertexArry[this->vertexNum].firstOut = nullptr;
    ++this->vertexNum;
}
```

