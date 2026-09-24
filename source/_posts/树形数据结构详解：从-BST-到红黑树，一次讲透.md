---
title: 树形数据结构详解：从 BST 到红黑树，一次讲透
date: 2026-09-24 09:11:22
tags:
  - tree
  - AVL
  - B-tree
  - Trie
categories: Data Structure
---

## 前言

树是计算机科学中最重要的一类数据结构。它以层次化的方式组织数据，在查找、插入、删除等操作上提供了远优于线性结构的效率。从数据库索引到文件系统，从编译器符号表到内存分配器，树无处不在。

但“树”并非单一结构，而是一个庞大的家族。二叉搜索树、AVL 树、红黑树、B 树、Treap、Splay、线段树、Trie……每一种树都针对特定场景做了取舍。

本文系统梳理各类树形结构的原理、性质、实现与适用场景，代码示例统一使用 C++17。

---

## 一、基础：二叉树与遍历

在讨论各种平衡树之前，先回顾二叉树的基本概念。

**二叉树**：每个节点最多有两个子节点，分别称为左孩子和右孩子。

```cpp
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
```

**四种遍历方式**：

| 遍历 | 顺序         | 典型用途               |
| ---- | ------------ | ---------------------- |
| 前序 | 根 → 左 → 右 | 复制树、序列化         |
| 中序 | 左 → 根 → 右 | BST 有序输出           |
| 后序 | 左 → 右 → 根 | 释放内存、计算子树信息 |
| 层序 | 逐层从左到右 | BFS、求深度            |

```cpp
void inorder(TreeNode* root, vector<int>& out) {
    if (!root) return;
    inorder(root->left, out);
    out.push_back(root->val);
    inorder(root->right, out);
}
```

**重要性质**：对二叉搜索树做中序遍历，得到的序列必然有序。这是所有平衡树的基础。

---

## 二、二叉搜索树（BST）

### 2.1 定义与性质

二叉搜索树（Binary Search Tree）满足：

- 左子树所有节点值 < 根节点值
- 右子树所有节点值 > 根节点值
- 左右子树也都是 BST

```cpp
TreeNode* search(TreeNode* root, int target) {
    while (root) {
        if (root->val == target) return root;
        root = (target < root->val) ? root->left : root->right;
    }
    return nullptr;
}
```

### 2.2 复杂度

| 操作 | 平均     | 最坏 |
| ---- | -------- | ---- |
| 查找 | O(log n) | O(n) |
| 插入 | O(log n) | O(n) |
| 删除 | O(log n) | O(n) |

**最坏情况**：按有序序列插入，BST 退化成链表，所有操作退化为 O(n)。

```
插入 1,2,3,4,5：
1
 \
  2
   \
    3
     \
      4
       \
        5
```

这就是平衡树诞生的原因——**防止树退化成链**。

---

## 三、AVL 树

### 3.1 核心思想

AVL 树是最早被发明的自平衡二叉搜索树（1962 年，Adelson-Velsky 和 Landis）。它的核心约束是：

> **任意节点的左右子树高度差（平衡因子）绝对值不超过 1。**

```cpp
struct AVLNode {
    int val, height;
    AVLNode *left, *right;
    AVLNode(int x) : val(x), height(1), left(nullptr), right(nullptr) {}
};

int height(AVLNode* n) { return n ? n->height : 0; }
int balanceFactor(AVLNode* n) { return n ? height(n->left) - height(n->right) : 0; }
void updateHeight(AVLNode* n) {
    n->height = 1 + max(height(n->left), height(n->right));
}
```

### 3.2 四种旋转

当插入或删除导致平衡因子超出 \[-1, 1\] 时，通过旋转恢复平衡。

**LL 型（左左）——右旋**

```
      z                y
     /                / \
    y        →       x   z
   /
  x
```

```cpp
AVLNode* rotateRight(AVLNode* z) {
    AVLNode* y = z->left;
    AVLNode* T2 = y->right;
    y->right = z;
    z->left = T2;
    updateHeight(z);
    updateHeight(y);
    return y;
}
```

**RR 型（右右）——左旋**

```
  z                    y
   \                  / \
    y        →       z   x
     \
      x
```

```cpp
AVLNode* rotateLeft(AVLNode* z) {
    AVLNode* y = z->right;
    AVLNode* T2 = y->left;
    y->left = z;
    z->right = T2;
    updateHeight(z);
    updateHeight(y);
    return y;
}
```

**LR 型**：先对左孩子左旋，再对根右旋。
**RL 型**：先对右孩子右旋，再对根左旋。

```cpp
AVLNode* insert(AVLNode* node, int val) {
    if (!node) return new AVLNode(val);

    if (val < node->val) node->left = insert(node->left, val);
    else if (val > node->val) node->right = insert(node->right, val);
    else return node;

    updateHeight(node);
    int bf = balanceFactor(node);

    // LL
    if (bf > 1 && val < node->left->val) return rotateRight(node);
    // RR
    if (bf < -1 && val > node->right->val) return rotateLeft(node);
    // LR
    if (bf > 1 && val > node->left->val) {
        node->left = rotateLeft(node->left);
        return rotateRight(node);
    }
    // RL
    if (bf < -1 && val < node->right->val) {
        node->right = rotateRight(node->right);
        return rotateLeft(node);
    }
    return node;
}
```

### 3.3 AVL 树的特点

| 维度       | 表现                     |
| ---------- | ------------------------ |
| 平衡严格度 | 高度差 ≤ 1，最严格       |
| 树高       | ≤ 1.44 log₂(n+2)         |
| 查找       | 最快（树最矮）           |
| 插入删除   | 旋转次数多，代价较高     |
| 适用场景   | **读多写少**，如静态字典 |

> AVL 树的平均查找效率优于红黑树，但插入删除时的旋转更频繁。因此，读写均衡的场景更倾向红黑树。

---

## 四、红黑树

### 4.1 核心思想

红黑树（Red-Black Tree）通过给节点着色来维持近似平衡，约束比 AVL 宽松，但插入删除时的调整次数更少。

**五条性质**：

1. 每个节点是红色或黑色。
2. 根节点是黑色。
3. 所有叶子节点（NIL 空节点）是黑色。
4. **红色节点的两个子节点必须是黑色**（不能有连续红节点）。
5. **从任一节点到其所有叶子的路径，包含相同数目的黑色节点**（黑高相同）。

由性质 4 和 5 可推出：**最长路径不超过最短路径的 2 倍**，因此树高 ≤ 2 log₂(n+1)。

```cpp
enum Color { RED, BLACK };

struct RBNode {
    int val;
    Color color;
    RBNode *left, *right, *parent;
    RBNode(int x) : val(x), color(RED), left(nullptr), right(nullptr), parent(nullptr) {}
};
```

### 4.2 插入调整

新插入的节点默认为**红色**，原因是不破坏性质 5（黑高不变），只需修正性质 4（红红冲突）。

插入后可能出现的冲突及处理：

**情况 1：父节点是黑色** → 无需调整。

**情况 2：父节点红色，叔叔节点红色** → 父与叔变黑，祖父变红，向上递归。

```
      G(黑)                G(红)
     /    \              /    \
   P(红)  U(红)   →    P(黑)  U(黑)
   /                    /
  N(红)                N(红)
```

**情况 3：父红叔黑，N 与父方向不同** → 先旋转父节点，转为情况 4。

**情况 4：父红叔黑，N 与父方向相同** → 父变黑，祖父变红，对祖父旋转。

```cpp
void insertFixup(RBNode*& root, RBNode* z) {
    while (z->parent && z->parent->color == RED) {
        RBNode* grandparent = z->parent->parent;
        if (z->parent == grandparent->left) {
            RBNode* uncle = grandparent->right;
            if (uncle && uncle->color == RED) {
                // 情况 2：叔叔红
                z->parent->color = BLACK;
                uncle->color = BLACK;
                grandparent->color = RED;
                z = grandparent;
            } else {
                if (z == z->parent->right) {
                    // 情况 3：LR
                    z = z->parent;
                    rotateLeft(root, z);
                }
                // 情况 4：LL
                z->parent->color = BLACK;
                grandparent->color = RED;
                rotateRight(root, grandparent);
            }
        } else {
            // 对称处理右侧
            RBNode* uncle = grandparent->left;
            if (uncle && uncle->color == RED) {
                z->parent->color = BLACK;
                uncle->color = BLACK;
                grandparent->color = RED;
                z = grandparent;
            } else {
                if (z == z->parent->left) {
                    z = z->parent;
                    rotateRight(root, z);
                }
                z->parent->color = BLACK;
                grandparent->color = RED;
                rotateLeft(root, grandparent);
            }
        }
    }
    root->color = BLACK;
}
```

> 删除调整涉及更多情况分支，核心思路是“兄弟借黑”，此处不再展开。完整实现可参考《算法导论》第 13 章。

### 4.3 红黑树 vs AVL 树

| 维度     | AVL 树                  | 红黑树                  |
| -------- | ----------------------- | ----------------------- |
| 平衡标准 | 高度差 ≤ 1              | 最长路径 ≤ 2 × 最短路径 |
| 树高     | ≤ 1.44 log n            | ≤ 2 log n               |
| 查找效率 | 略优                    | 略低                    |
| 插入旋转 | O(log n) 次（可能连锁） | **最多 2 次**           |
| 删除旋转 | O(log n) 次             | **最多 3 次**           |
| 着色操作 | 无                      | O(log n) 次变色         |
| 适用场景 | 读多写少                | **读写均衡**            |

红黑树的优势在于插入删除时的**结构调整次数恒定**，因此成为大多数标准库 `map`、`set` 的底层实现。

**实际应用**：

- C++ STL：`std::map`、`std::set`、`std::multimap`、`std::multiset`
- Java：`TreeMap`、`TreeSet`
- Linux 内核：CFS 调度器、`epoll` 就绪队列、虚拟内存管理

---

## 五、B 树与 B+ 树

### 5.1 为什么需要多路搜索树

AVL 和红黑树都是**二叉**结构。当数据量达到百万级时，树高约 20 层；达到十亿级时，树高约 30 层。每次查找需要 30 次节点访问——如果节点在磁盘上，这意味着 30 次磁盘 I/O，代价极高。

B 树的核心思想：**一个节点存多个键，有多个孩子，大幅降低树高。**

### 5.2 B 树

一棵 m 阶 B 树满足：

- 每个节点最多 m 个孩子，m-1 个键。
- 除根节点外，每个节点至少 ⌈m/2⌉ 个孩子。
- 所有叶子在同一层。

```
                  [17 | 35]
                /     |     \
        [5|11]   [23|29]   [41|47]
```

### 5.3 B+ 树

B+ 树是 B 树的变体，主要区别：

1. **所有数据只存在叶子节点**，内部节点仅存索引键。
2. **叶子节点通过链表串联**，支持高效范围查询。
3. 内部节点的键是子树的**最大值或最小值**（冗余存储）。

```
              [17 | 35]
            /     |     \
     [5|11] → [23|29] → [41|47]
       ↓        ↓         ↓
     数据      数据       数据
```

**B+ 树 vs B 树**：

| 维度     | B 树                 | B+ 树            |
| -------- | -------------------- | ---------------- |
| 数据存储 | 所有节点             | 仅叶子节点       |
| 范围查询 | 需中序遍历，跨层跳转 | 叶子链表顺序扫描 |
| 内部节点 | 存键值对             | 仅存索引         |
| 单点查询 | 可能在内部命中，更快 | 必须到叶子       |
| 应用     | 文件系统             | **数据库索引**   |

**实际应用**：

- MySQL InnoDB 索引（B+ 树）
- PostgreSQL 索引
- NTFS、ext4 等文件系统（B 树或 B+ 树变体）

---

## 六、Treap

### 6.1 核心思想

Treap = Tree + Heap。它同时满足：

- **BST 性质**：按 `val` 排序。
- **堆性质**：按随机优先级 `priority` 构成大顶堆（或小顶堆）。

由于优先级是随机的，Treap 的期望高度为 O(log n)，且实现比红黑树简单得多。

```cpp
struct TreapNode {
    int val, priority, size;
    TreapNode *left, *right;
    TreapNode(int v) : val(v), priority(rand()), size(1), left(nullptr), right(nullptr) {}
};
```

### 6.2 旋转实现

Treap 通过旋转维持堆性质：

```cpp
void rotateRight(TreapNode*& p) {
    TreapNode* q = p->left;
    p->left = q->right;
    q->right = p;
    p = q;
}

void insert(TreapNode*& p, int val) {
    if (!p) { p = new TreapNode(val); return; }
    if (val < p->val) {
        insert(p->left, val);
        if (p->left->priority > p->priority) rotateRight(p);
    } else {
        insert(p->right, val);
        if (p->right->priority > p->priority) rotateLeft(p);
    }
}
```

### 6.3 特点

| 维度           | 表现                                |
| -------------- | ----------------------------------- |
| 实现难度       | 比红黑树简单得多                    |
| 期望高度       | O(log n)                            |
| 最坏情况       | O(n)（概率极低）                    |
| 是否支持持久化 | 支持（无需旋转的 split/merge 版本） |
| 适用场景       | 竞赛、需要持久化的有序集合          |

**扩展**：非旋转 Treap（FHQ Treap）使用 `split` 和 `merge` 两个操作即可实现所有功能，代码更简洁，是竞赛中的首选。

---

## 七、Splay 树

### 7.1 核心思想

Splay 树（伸展树）不存储任何平衡信息。每次访问一个节点后，通过 **splay 操作**将该节点旋转到根。

关键性质：**均摊复杂度为 O(log n)**，尽管单次操作最坏可能 O(n)。

### 7.2 Splay 操作

将节点 x 旋转到根，分三种情况：

- **Zig**：父节点是根，单旋。
- **Zig-Zig**：x 和父节点同为左孩子或同为右孩子，先旋转父节点，再旋转 x。
- **Zig-Zag**：x 和父节点方向不同，先旋转 x，再旋转 x。

```cpp
void splay(Node*& root, Node* x) {
    while (x->parent) {
        Node* p = x->parent;
        Node* g = p->parent;
        if (!g) {
            rotate(root, x);
        } else if ((x == p->left) == (p == g->left)) {
            rotate(root, p);
            rotate(root, x);
        } else {
            rotate(root, x);
            rotate(root, x);
        }
    }
}
```

### 7.3 特点

| 维度       | 表现                               |
| ---------- | ---------------------------------- |
| 均摊复杂度 | O(log n)                           |
| 单次最坏   | O(n)                               |
| 局部性     | 最近访问的节点靠近根，**缓存友好** |
| 适用场景   | 频繁访问局部数据（如 LRU 类场景）  |

**经典应用**：LCT（Link-Cut Tree）的辅助树。

---

## 八、线段树与树状数组

前面讨论的都是“有序集合”型树，接下来是“区间查询”型树。

### 8.1 线段树

线段树将区间递归二分为树，每个节点维护一个区间的信息（和、最值、积等），支持 O(log n) 的区间查询与修改。

```cpp
struct SegTree {
    vector<int> tree;
    int n;

    SegTree(vector<int>& nums) {
        n = nums.size();
        tree.assign(4 * n, 0);
        build(nums, 1, 0, n - 1);
    }

    void build(vector<int>& nums, int node, int l, int r) {
        if (l == r) { tree[node] = nums[l]; return; }
        int mid = (l + r) / 2;
        build(nums, node * 2, l, mid);
        build(nums, node * 2 + 1, mid + 1, r);
        tree[node] = tree[node * 2] + tree[node * 2 + 1];
    }

    void update(int node, int l, int r, int idx, int val) {
        if (l == r) { tree[node] = val; return; }
        int mid = (l + r) / 2;
        if (idx <= mid) update(node * 2, l, mid, idx, val);
        else update(node * 2 + 1, mid + 1, r, idx, val);
        tree[node] = tree[node * 2] + tree[node * 2 + 1];
    }

    int query(int node, int l, int r, int ql, int qr) {
        if (qr < l || r < ql) return 0;
        if (ql <= l && r <= qr) return tree[node];
        int mid = (l + r) / 2;
        return query(node * 2, l, mid, ql, qr) +
               query(node * 2 + 1, mid + 1, r, ql, qr);
    }
};
```

### 8.2 树状数组

树状数组（Fenwick Tree / BIT）以更小的常量和更短的代码实现前缀和查询与单点更新。

```cpp
class Fenwick {
    vector<int> bit;
    int n;
public:
    Fenwick(int n) : n(n), bit(n + 1, 0) {}

    void update(int i, int delta) {
        for (++i; i <= n; i += i & -i) bit[i] += delta;
    }

    int query(int i) {  // [0, i] 前缀和
        int sum = 0;
        for (++i; i > 0; i -= i & -i) sum += bit[i];
        return sum;
    }
};
```

### 8.3 对比

| 维度     | 线段树               | 树状数组         |
| -------- | -------------------- | ---------------- |
| 代码长度 | 长                   | 短               |
| 常数     | 较大                 | 小               |
| 支持操作 | 区间和、最值、懒标记 | 前缀和、单点更新 |
| 区间最值 | 支持                 | 不支持           |
| 可扩展性 | 强                   | 有限             |
| 适用场景 | 复杂区间问题         | 前缀和类问题     |

---

## 九、Trie（前缀树）

Trie 用于存储字符串集合，支持 O(m) 的前缀查询（m 为串长）。

```cpp
struct TrieNode {
    TrieNode* children[26];
    bool isEnd;
    TrieNode() : isEnd(false) {
        fill(children, children + 26, nullptr);
    }
};

class Trie {
    TrieNode* root;
public:
    Trie() : root(new TrieNode()) {}

    void insert(const string& word) {
        TrieNode* node = root;
        for (char c : word) {
            int idx = c - 'a';
            if (!node->children[idx]) node->children[idx] = new TrieNode();
            node = node->children[idx];
        }
        node->isEnd = true;
    }

    bool search(const string& word) {
        TrieNode* node = root;
        for (char c : word) {
            int idx = c - 'a';
            if (!node->children[idx]) return false;
            node = node->children[idx];
        }
        return node->isEnd;
    }

    bool startsWith(const string& prefix) {
        TrieNode* node = root;
        for (char c : prefix) {
            int idx = c - 'a';
            if (!node->children[idx]) return false;
            node = node->children[idx];
        }
        return true;
    }
};
```

**应用**：自动补全、拼写检查、IP 路由（二进制 Trie）、敏感词过滤。

---

## 十、总览对比

| 树类型   | 平衡方式        | 查找          | 插入          | 删除          | 典型应用     |
| -------- | --------------- | ------------- | ------------- | ------------- | ------------ |
| BST      | 无              | O(n) 最坏     | O(n)          | O(n)          | 教学         |
| AVL      | 高度差 ≤ 1      | O(log n)      | O(log n)      | O(log n)      | 读多写少     |
| 红黑树   | 颜色约束        | O(log n)      | O(log n)      | O(log n)      | STL map/set  |
| B 树     | 多路平衡        | O(log n)      | O(log n)      | O(log n)      | 文件系统     |
| B+ 树    | 多路平衡 + 叶链 | O(log n)      | O(log n)      | O(log n)      | 数据库索引   |
| Treap    | 随机优先级      | 期望 O(log n) | 期望 O(log n) | 期望 O(log n) | 竞赛、持久化 |
| Splay    | 访问即上浮      | 均摊 O(log n) | 均摊 O(log n) | 均摊 O(log n) | LCT、缓存    |
| 线段树   | 完全二叉树      | O(log n) 区间 | O(log n)      | O(log n)      | 区间查询     |
| 树状数组 | 二进制索引      | O(log n)      | O(log n)      | O(log n)      | 前缀和       |
| Trie     | 前缀共享        | O(m)          | O(m)          | O(m)          | 字符串检索   |

---

## 十一、如何选择

1. **需要有序集合，读写均衡** → 红黑树（`std::map`、`std::set`）。
2. **需要有序集合，读远多于写** → AVL 树。
3. **数据量大，需持久化到磁盘** → B+ 树。
4. **需要区间查询/修改** → 线段树或树状数组。
5. **需要前缀匹配** → Trie。
6. **竞赛中需要可持久化或简洁实现** → FHQ Treap。
7. **需要动态维护连通性** → 并查集（本质是树）。
8. **需要高效范围查询的数据库索引** → B+ 树。

---

## 十二、总结

树形结构的演化史，本质上是一部**在查找、插入、删除三种操作之间寻找平衡**的历史：

- **BST** 提供了有序性，但可能退化。
- **AVL** 用严格平衡换取最优查找，牺牲写性能。
- **红黑树** 放宽平衡条件，用更少的旋转换取均衡的读写性能，成为工业界最广泛使用的平衡树。
- **B/B+ 树** 通过多路设计降低树高，适配磁盘 I/O，成为数据库的基石。
- **Treap / Splay** 用随机化或均摊分析，以更简单的实现达到对数复杂度。
- **线段树 / 树状数组** 将树的思想应用于区间问题，是算法竞赛的核心工具。
- **Trie** 用空间换时间，将字符串检索降到与串长相关。

一句话概括：

> **没有最好的树，只有最适合场景的树。理解每种树的约束与代价，才能在工程中做出正确的选择。**
