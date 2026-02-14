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

## 最短距离

### 单源最短距离

寻找最短路径是图论中的一个经典问题。给边指定某个权，可以表示城市间的距离，任务执行的时间段，两地之间信息传输的开销，两地之间物质运输的总量等。当确定从顶点v到顶点u的最短路径时，必须记录中间顶点w的相关距离信息。该信息可记录为与顶点相关的标记，此标记只表示v到w的距离，或者该路径中从w的前驱到v的距离。查找最短路径的方法需要这些标记。根据这些标记的更新次数，解决最短路径问题的方法分为两类：标记设置法(label-setting)和标记校正法(label-correcting).

应用场景以及局限性：标记设置法需要处理遍历经过的每一个顶点，给每个顶点设置一个值，该值一直到运行结束都保持不变，但此方法只能处理权值为正的图。第二类方法是标记校正法，使用该方法时允许修改标记。这种方法可以运用于权值为负但不含反向循环的图（反向环是指构成此环的边的权相加为一个负值），但该方法可以保证对所有的顶点而言，图处理完后当前距离为最短路径。然而大多数标记设置法和标记校正法都可以归纳为同一类，因为它们都可以找到从一个顶点到其他所有顶点间的最短路径

#### **通用算法**

```c++
genericShortestPathAlogrithm(带权的简单有向图digraph, 顶点first)
	for 所有顶点v
		currDist(v)=∞;
	currDist(first)=0;
	初始化toBeChecked;
	while toBeChecked非空:
		v=toBeChecked中的一个顶点；
		从toBeChecked中删除v;
		for v的所有邻接顶点u
			if currDist(u)> currDist(v)+ weight(edge(vu))
			currDist(u)= currDist(v)+ weight(edge(vu));
			predecessor(u)=v;
			如果u不在toBeChecked中，将u添加到其中。
```

**算法解读**

该算法的核心思想是 **“松弛” (Relaxation)**。它维护一个从源点 `first` 到图中每个顶点 `v` 的当前已知最短距离估计值，我们称之为 `currDist(v)`。算法通过不断迭代，逐步优化这些距离估计值，直到它们收敛为真正的最短路径长度。

1. **初始化**:
   - 将源点 `first` 到自身的距离 `currDist(first)` 初始化为 0。
   - 将源点到所有其他顶点的距离 `currDist(v)` 初始化为无穷大（∞），表示这些顶点暂时不可达。
   - 同时，维护一个前驱顶点数组 `predecessor(v)`，用于记录路径，初始化为空。
2. **迭代与松弛**:
   - 算法维护一个待处理的顶点集合 `toBeChecked`。
   - 在每次迭代中，从 `toBeChecked` 中取出一个顶点 `v` 进行处理。
   - 对于 `v` 的每一个**出边**所指向的邻接顶点 `u`，进行一次“松弛”操作：
     - **判断**：如果 “通过 `v` 到达 `u` 的路径” (`currDist(v) + weight(v, u)`) 比 “当前已知的到达 `u` 的最短路径” (`currDist(u)`) 更短。
     - **更新**：如果更短，就更新 `currDist(u)` 为这个更短的值，并记录 `u` 的前驱是 `v`（即 `predecessor(u) = v`）。这代表我们找到了一条到达 `u` 的更优路径。
3. **终止**:
   - 当 `toBeChecked` 集合为空时，算法终止。此时，`currDist` 数组中存储的就是从源点 `first` 到各顶点的最短路径长度。

#### **在我的十字链表中的应用**

```c++
#pragma once
#include <iostream>
#include <vector>      // 新增：用于存储结果
#include <queue>       // 新增：用于优先队列
#include <limits>      // 新增：用于表示无穷大
#include <functional>  // 新增：用于优先队列的比较器

using namespace std;

// ... (EdgeNode 和 VertexNode 的定义保持不变) ...

// 十字链表有向图
template <class DataType, class WeightType>
class DiGraph
{
public:
    // ... (现有构造函数、析构函数和成员函数) ...
    void print();
    int outDegree(const DataType data);
    int inDegree(const DataType data);

    // 新增：Dijkstra 算法的返回结果结构体
    struct DijkstraResult {
        vector<WeightType> dist; // 存储从源点到各顶点的最短距离
        vector<int> path;        // 存储最短路径的前驱顶点索引
    };

    // 新增：Dijkstra 算法的声明
    DijkstraResult dijkstra(const DataType startVertex);


private:
    // ... (现有私有成员) ...
};

// ... (现有函数实现，如构造函数, addEdge, print 等) ...
```

```c++
// ... (所有现有函数实现) ...

// 新增：Dijkstra 算法的实现
template <class DataType, class WeightType>
typename DiGraph<DataType, WeightType>::DijkstraResult
DiGraph<DataType, WeightType>::dijkstra(const DataType startVertex)
{
    // 1. 初始化
    int startIndex = search(startVertex);
    if (startIndex == -1)
    {
        cerr << "Dijkstra Error: Start vertex not found in the graph.\n";
        return {}; // 返回空结果
    }

    DijkstraResult result;
    // 初始化距离数组为无穷大
    result.dist.assign(this->capacity, numeric_limits<WeightType>::max());
    // 初始化路径数组为-1（表示没有前驱）
    result.path.assign(this->capacity, -1);

    // 设置源点的距离为0
    result.dist[startIndex] = 0;

    // 定义优先队列，用于存储 {距离, 顶点索引} 对。
    // `greater` 使其成为最小优先队列（距离最小的在顶部）。
    using Pair = pair<WeightType, int>;
    priority_queue<Pair, vector<Pair>, greater<Pair>> pq;

    // 将源点加入优先队列
    pq.push({0, startIndex});

    // 2. 主循环 (while toBeChecked 非空)
    while (!pq.empty())
    {
        // 3. v = toBeChecked 中 currDist(v) 最小的顶点
        WeightType currentDist = pq.top().first;
        int v_idx = pq.top().second;
        pq.pop(); // 从 toBeChecked 中删除 v

        // 如果我们已经找到了到 v_idx 的更短路径，则跳过此次处理
        if (currentDist > result.dist[v_idx])
        {
            continue;
        }

        // 4. for v 的所有邻接顶点 u
        EdgeNode<WeightType> *edge = this->vertexArry[v_idx].firstOut;
        while (edge != nullptr)
        {
            int u_idx = edge->tailIndex;
            WeightType weight = edge->weight;

            // 5. 松弛操作 (Relaxation)
            // if currDist(u) > currDist(v) + weight(edge(vu))
            if (result.dist[u_idx] > result.dist[v_idx] + weight)
            {
                // 更新距离
                result.dist[u_idx] = result.dist[v_idx] + weight;
                // 更新前驱
                result.path[u_idx] = v_idx;
                // 将更新后的顶点 u 加入优先队列
                pq.push({result.dist[u_idx], u_idx});
            }
            edge = edge->headEdge;
        }
    }

    return result;
}
```

**代码知识解读**

1. 初始化部分

```c++
int startIndex = search(startVertex);
if (startIndex == -1)
{
    cerr << "Dijkstra Error: Start vertex not found in the graph.\n";
    return {}; // 返回空结果
}

DijkstraResult result;
// 初始化距离数组为无穷大
result.dist.assign(this->capacity, numeric_limits<WeightType>::max());
// 初始化路径数组为-1（表示没有前驱）
result.path.assign(this->capacity, -1);

// 设置源点的距离为0
result.dist[startIndex] = 0;
```

- `result.dist` 和 `result.path` 都是 `vector` 容器：

  - `vector::assign(n, val)`：是 `vector` 的初始化方法，意思是 “给容器分配 `n` 个元素，每个元素的值都是 `val`”；
  - 这里 `this->capacity` 是图的顶点总数，所以 `dist` 数组初始时所有顶点到起点的距离都是 “无穷大”（`numeric_limits<WeightType>::max()`），`path` 数组初始时所有顶点都没有前驱（用 `-1` 标记）；
  - 最后把起点自身的距离设为 0（到自己的最短路径长度为 0）。

  

2. 优先队列（核心容器）的定义与初始化

```c++
// 定义优先队列，用于存储 {距离, 顶点索引} 对。
// `greater` 使其成为最小优先队列（距离最小的在顶部）。
using Pair = pair<WeightType, int>;
priority_queue<Pair, vector<Pair>, greater<Pair>> pq;

// 将源点加入优先队列
pq.push({0, startIndex});
```

这部分是你重点要掌握的，我拆成 `pair` 和 `priority_queue` 两部分讲：

(1)  `pair` 详解

- **本质**：`pair` 是 C++ STL 中的一个模板类，定义在 `<utility>` 头文件中，作用是**把两个不同类型的值 “打包” 成一个整体**，可以理解为 “极简版的结构体”。

- **语法**：`pair<T1, T2>`，其中 `T1` 是第一个元素的类型，`T2` 是第二个元素的类型；

  - 访问元素：`pair.first` 取第一个元素，`pair.second` 取第二个元素；
  - 初始化：可以用 `{val1, val2}`（C++11 及以上）或 `make_pair(val1, val2)`。

  

- **代码中的作用**：`Pair = pair<WeightType, int>` 表示 “一个权重（距离） + 一个顶点索引” 的组合，目的是把 “到某个顶点的距离” 和 “顶点编号” 绑定在一起，方便优先队列排序。

（2）`priority_queue`（优先队列）详解

- **本质**：优先队列是一种 “特殊的队列”，队列的特点是 “先进先出”，但优先队列会**根据元素的优先级自动排序**，每次取出的都是优先级最高的元素（默认是最大的元素）。

- **语法**：`priority_queue<元素类型, 底层容器类型, 比较规则>`：

  - 第一个参数：队列中存储的元素类型（这里是我们定义的 `Pair`）；

  - 第二个参数：优先队列的底层实现容器（必须是顺序容器，默认是 `vector`，所以这里写 `vector<Pair>` 是显式指定，和默认一致）；

  - 第三个参数：比较规则（决定优先级）：

    - 默认：`less<T>` → 最大优先队列（数值大的元素优先级高，堆顶是最大值）；
    - `greater<T>` → 最小优先队列（数值小的元素优先级高，堆顶是最小值）。

    

  

- **代码中的作用**：

  - 我们需要每次找到 “当前距离起点最近的未处理顶点”，所以必须用**最小优先队列**（`greater<Pair>`）；
  - 优先队列排序规则：`pair` 的比较是**先比第一个元素，第一个相等再比第二个**。这里 `Pair` 的第一个元素是距离（`WeightType`），所以队列会按 “距离从小到大” 排序，堆顶永远是当前距离起点最近的顶点。

  

- **核心操作**：

  - `pq.push({0, startIndex})`：把起点（距离 0，索引 `startIndex`）加入队列；
  - `pq.top()`：获取堆顶元素（不删除）；
  - `pq.pop()`：删除堆顶元素。

  

3. 主循环（Dijkstra 核心逻辑）

```c++
while (!pq.empty())
{
    // 取出当前距离最小的顶点
    WeightType currentDist = pq.top().first; // 取pair的第一个元素（距离）
    int v_idx = pq.top().second; // 取pair的第二个元素（顶点索引）
    pq.pop(); // 从队列中删除该顶点

    // 跳过：如果当前记录的距离比已知的最短距离大，说明是过时的信息
    if (currentDist > result.dist[v_idx])
    {
        continue;
    }

    // 遍历当前顶点v的所有邻接边
    EdgeNode<WeightType> *edge = this->vertexArry[v_idx].firstOut;
    while (edge != nullptr)
    {
        int u_idx = edge->tailIndex; // 邻接顶点u的索引
        WeightType weight = edge->weight; // 边v→u的权重

        // 松弛操作：如果经过v到u的路径更短，就更新
        if (result.dist[u_idx] > result.dist[v_idx] + weight)
        {
            result.dist[u_idx] = result.dist[v_idx] + weight; // 更新距离
            result.path[u_idx] = v_idx; // 记录u的前驱是v
            pq.push({result.dist[u_idx], u_idx}); // 把更新后的u加入队列
        }
        edge = edge->headEdge; // 遍历下一条邻接边
    }
}
```

- **关键容器操作**：
  1. `pq.top().first` / `pq.top().second`：取出堆顶 `pair` 的两个元素（距离和顶点索引），这是 `pair` 最核心的访问方式；
  2. 松弛操作后 `pq.push({result.dist[u_idx], u_idx})`：把更新后的（新距离，顶点 u）加入优先队列 —— 即使 u 已经在队列中，也不需要删除旧的，后续处理时发现 `currentDist > result.dist[v_idx]` 会直接跳过，这是 Dijkstra 用优先队列实现的常规写法；
  3. `result.dist` 和 `result.path` 作为 `vector`，通过下标 `[u_idx]` 直接访问和修改元素，这是 `vector` 最常用的操作（随机访问）。

##### **关键代码解读**

`currentDist > result.dist[v_idx] 会直接跳过，这是 Dijkstra 用优先队列实现的常规写法`

当 `currentDist > result.dist[v_idx]` 时要跳过当前顶点的处理，这确实是 Dijkstra 算法用优先队列实现时最容易困惑的点#

###### 核心原因先讲结论

优先队列中**可能存在同一个顶点的多个 “旧 / 过时” 记录**，`currentDist > result.dist[v_idx]` 这个判断就是用来过滤这些无效记录的 —— 当我们从队列中取出某个顶点时，如果它记录的距离已经不是我们已知的最短距离了，说明这个记录已经没用了，直接跳过即可。

###### 用具体例子拆解逻辑

假设我们的图有顶点 A (0)、B (1)、C (2)，边的权重如下：

- A→B：权重 5
- A→C：权重 10
- B→C：权重 1

步骤 1：初始化

- 起点是 A (0)，`result.dist = [0, ∞, ∞]`（∞ 表示无穷大）
- 优先队列 `pq` 中加入 `(0, 0)`（距离 0，顶点 0）

步骤 2：第一次处理队列

- 取出队首 `(0, 0)`（currentDist=0，v_idx=0），`0 == result.dist[0]`，不跳过；

- 遍历 A 的邻接边：

  - A→B：更新 `result.dist[1] = 0+5=5`，队列加入 `(5, 1)`；
  - A→C：更新 `result.dist[2] = 0+10=10`，队列加入 `(10, 2)`；

  

- 此时队列中有 `(5, 1)`、`(10, 2)`。

步骤 3：第二次处理队列

- 取出队首 `(5, 1)`（currentDist=5，v_idx=1），`5 == result.dist[1]`，不跳过；

- 遍历 B 的邻接边：

  - B→C：原来 `result.dist[2] = 10`，现在 `5+1=6 < 10`，所以更新 `result.dist[2] = 6`，队列加入 `(6, 2)`；

  

- 此时队列中有 `(6, 2)`、`(10, 2)`（注意：C 顶点有两个记录了！）。

步骤 4：第三次处理队列

- 取出队首 `(6, 2)`（currentDist=6，v_idx=2），`6 == result.dist[2]`，不跳过；
- 遍历 C 的邻接边（假设无），处理结束；
- 此时队列中还剩 `(10, 2)`。

步骤 5：第四次处理队列（关键！）

- 取出队首 `(10, 2)`（currentDist=10，v_idx=2）；

- 此时检查：`10 > result.dist[2] (6)` → 条件成立，直接跳过；

- 为什么要跳过？

  - 这个 `(10, 2)` 是**早期的旧记录**（A→C 直接走的距离），但我们已经找到了更短的路径（A→B→C，距离 6），并且已经处理过 C 顶点了；
  - 如果不跳过，会重复处理 C 的邻接边（即使没有），甚至可能错误地更新距离（但这里已经是最短了，不会更新），纯粹浪费计算资源。

  

###### 为什么队列中会出现重复的顶点记录？

因为 Dijkstra 算法的**松弛操作**：每当我们找到一个顶点的更短路径时，都会把 “新距离 + 顶点” 重新加入队列，而不会删除队列中已有的 “旧距离 + 顶点”。

- 优先队列的特性是 “只能取堆顶、不能删除中间元素”，所以无法高效删除旧记录；
- 与其费力删除，不如在取出时做一次判断：如果当前记录的距离比已知的最短距离大，说明这是过时的，直接跳过即可 —— 这是工程上 “空间换时间” 的最优选择。
