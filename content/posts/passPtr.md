---
title: PassPtr
published: 2026-02-01T10:46:09+08:00
summary: "辨析传递指针还是指针的引用"
cover:
  image: https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202601300127015.png
tags: [关于传递指针还是指针引用]
categories: '知识点'
draft: false
lang: ''
---

## 知识讲解

### 一、需要传递指针的引用（`T*&`）的场景

核心判断：**函数需要修改 “指针变量本身的值”（即改变指针指向的内存地址）**。

指针的引用本质是原指针的 “别名”，函数内对它的修改会直接作用于外部的原指针，常见场景包括：

#### 1. 链表 / 树的结构修改（需要改变指针指向）

- 链表头插 / 尾插（尤其是空链表插入第一个节点）：需要修改头指针 / 尾指针的指向；
- 二叉树节点删除（如你之前看到的 `deleteByMerging`）：需要修改父节点指向待删节点的指针；
- 链表节点删除（删除头节点或中间节点后衔接前后指针）：需要改变前驱节点的 `next` 指针指向；
- 空二叉树插入根节点：需要把外部的 `root` 指针从 `nullptr` 改为新节点地址。

**示例**：链表头插必须用 `Node*&`，否则无法修改原头指针：



```
// 正确：传指针引用，修改原头指针
void insertHead(Node*& head, int val) {
    Node* newNode = new Node(val);
    newNode->next = head;
    head = newNode; // 直接修改外部的head指针指向
}

// 错误：传普通指针，仅修改函数内副本
void insertHeadWrong(Node* head, int val) {
    Node* newNode = new Node(val);
    newNode->next = head;
    head = newNode; // 仅修改副本，外部head不变
}
```

#### 2. 函数内给外部指针赋值（初始化指针）

当函数需要创建对象，并让外部的指针指向这个新对象时，必须传指针引用：



```
// 初始化空指针为新节点
void createNode(Node*& p, int val) {
    p = new Node(val); // 外部的p会直接指向新节点
}

int main() {
    Node* ptr = nullptr;
    createNode(ptr, 10); // 传引用后，ptr从nullptr变为指向新节点
    cout << ptr->val; // 输出10，生效
    return 0;
}
```

### 二、直接传递普通指针（`T*`）的场景

核心判断：**函数仅操作指针指向的 “内容”，不需要改变指针本身的指向**。

普通指针传参是 “值传递”，函数内拿到指针的副本，但副本和原指针指向同一个对象，因此可以操作对象内容，只是不能改指针本身的指向。

#### 1. 仅访问 / 读取指针指向的内容

比如遍历链表、打印二叉树节点值、查询节点值等：



```
// 遍历链表：仅读取节点值，无需改指针指向
void traverse(Node* head) {
    Node* cur = head;
    while (cur != nullptr) {
        cout << cur->val << " "; // 仅访问内容
        cur = cur->next; // 改的是函数内的cur副本，不影响外部head
    }
}
```

#### 2. 修改指针指向的对象的成员（不改变指针指向）

比如修改链表节点的 `val`、二叉树节点的 `data` 等：



```
// 修改节点值：仅改对象成员，无需改指针指向
void modifyNode(Node* p, int newVal) {
    if (p != nullptr) {
        p->val = newVal; // 操作的是指针指向的对象，生效
    }
}

int main() {
    Node* node = new Node(5);
    modifyNode(node, 10); // 传普通指针，node->val变为10
    cout << node->val; // 输出10
    return 0;
}
```

#### 3. 函数内仅临时使用指针（不影响外部）

比如判断节点是否存在、计算链表长度等，仅用指针遍历，不修改任何指针指向：



```
// 计算链表长度：仅遍历，无需改指针
int getLength(Node* head) {
    int len = 0;
    Node* cur = head;
    while (cur != nullptr) {
        len++;
        cur = cur->next;
    }
    return len;
}
```

### 总结

1. **传指针的引用（`T\*&`）**：当需要**改变指针本身的指向**（修改指针变量存储的地址值）时使用，比如链表 / 树的结构修改、初始化空指针；
2. **传普通指针（`T\*`）**：当仅需要**操作指针指向的对象内容**（读 / 改成员），不需要改变指针指向时使用，比如遍历、修改节点值；
3. 核心记忆：指针的引用改 “指针本身”，普通指针改 “指针指向的内容”。

简单类比：普通指针传参是 “拿复印件办事”（改复印件没用），指针引用传参是 “拿原件办事”（改原件直接生效）。

## 样例实战

在BSTree代码中

#### 1.关于turn函数的参数

![image-20260201130250273](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202602011302887.png)

#### 2.迭代插入函数

![image-20260201130354555](https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202602011303054.png)