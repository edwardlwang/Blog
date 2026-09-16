---
title: 贪心算法（Greedy Algorithm）入门到进阶
date: 2026-09-16 16:20:11
tags:
  - Greedy
  - 贪心
categories: Algorithm
---

## 前言

贪心算法（Greedy Algorithm）是一种在每一步决策中都选择**当前看起来最优**的策略，并期望通过一系列局部最优选择得到全局最优解的算法思想。

它的代码通常简短、直观，时间复杂度低，因此在竞赛和工程中应用广泛。但贪心法也有一个致命弱点：**局部最优不一定能推出全局最优**。因此，判断一个问题能否用贪心求解，以及如何证明贪心的正确性，是掌握贪心法的关键。

本文系统梳理贪心法的适用条件、证明方法、经典模型与常见陷阱，代码示例统一使用 C++17。

---

## 一、什么是贪心法

贪心法的基本流程：

1. 将问题分解为一系列决策步骤。
2. 在每一步，根据某个**贪心策略**选择当前最优的选项。
3. 一旦做出选择，就**不再回溯**，继续下一步。
4. 最终得到的解即为算法的输出。

与动态规划不同，贪心法不保存子问题的解，也不比较多种决策路径，因此实现更简单、效率更高。

**典型例子：活动选择问题**

给定一组活动，每个活动有开始时间和结束时间，要求选择尽可能多的互不重叠的活动。

贪心策略：**优先选择结束时间最早的活动**。

```cpp
#include <vector>
#include <algorithm>
using namespace std;

int activitySelection(vector<pair<int, int>>& intervals) {
    // intervals[i] = {start, end}
    sort(intervals.begin(), intervals.end(),
         [](const auto& a, const auto& b) { return a.second < b.second; });

    int count = 0;
    int lastEnd = INT_MIN;
    for (const auto& [start, end] : intervals) {
        if (start >= lastEnd) {
            ++count;
            lastEnd = end;
        }
    }
    return count;
}
```

该策略可以证明是最优的（见第四节）。

---

## 二、贪心法的适用条件

一个问题能用贪心法求解，通常需要满足以下两个性质：

### 1. 贪心选择性质（Greedy Choice Property）

**全局最优解可以通过一系列局部最优选择达到。**

也就是说，在每一步做出贪心选择后，原问题可以转化为一个规模更小的子问题，且该选择一定包含在某个全局最优解中。

### 2. 最优子结构（Optimal Substructure）

**问题的最优解包含其子问题的最优解。**

这一点与动态规划相同。但贪心法额外要求：子问题的最优解可以通过贪心选择直接得到，而不需要比较多个候选。

> 注意：最优子结构是必要条件，但不是充分条件。许多问题具有最优子结构，却无法用贪心法求解（如 0-1 背包）。

---

## 三、贪心法 vs 动态规划

| 维度       | 贪心法                                   | 动态规划                     |
| ---------- | ---------------------------------------- | ---------------------------- |
| 决策方式   | 每步选当前最优，不回溯                   | 枚举所有可能决策，取全局最优 |
| 子问题     | 只求解一个子问题                         | 求解多个重叠子问题           |
| 状态保存   | 不需要                                   | 需要保存子问题结果           |
| 时间复杂度 | 通常较低（常为 $O(n \log n)$ 或 $O(n)$） | 通常较高（多项式级）         |
| 正确性     | 依赖贪心选择性质的证明                   | 依赖最优子结构和无后效性     |
| 典型问题   | 活动选择、霍夫曼编码、最小生成树         | 背包、LCS、编辑距离          |

**关键区别**：贪心法“目光短浅”，只考虑当前；动态规划“深谋远虑”，比较所有可能。

例如，分数背包问题可以用贪心法（按性价比排序），但 0-1 背包问题不能，因为物品不可分割，贪心选择可能导致无法装入剩余空间。

---

## 四、证明贪心正确性的方法

贪心法最困难的部分不是写代码，而是**证明贪心策略确实能得到全局最优解**。常用方法有以下几种：

### 1. 交换论证（Exchange Argument）

**思路**：假设存在一个最优解 $O$，如果 $O$ 与贪心解 $G$ 不同，则可以通过交换 $O$ 中的某些选择，使其逐步逼近 $G$，且不降低解的质量。最终证明 $G$ 也是最优解。

**示例：活动选择问题**

设贪心法选择的第一个活动为 $g_1$（结束时间最早）。设某个最优解为 $O = \{o_1, o_2, \dots\}$，其中 $o_1$ 是 $O$ 中结束时间最早的活动。由于 $g_1$ 是所有活动中结束时间最早的，因此 $end(g_1) \le end(o_1)$。

将 $O$ 中的 $o_1$ 替换为 $g_1$，由于 $g_1$ 结束更早，不会与 $O$ 中后续活动冲突，因此替换后的解仍然合法，且活动数量不变。于是存在一个包含 $g_1$ 的最优解。对剩余子问题归纳，即可证明贪心解最优。

### 2. 归纳法

证明贪心法前 $k$ 步的选择可以扩展为某个全局最优解。通常与交换论证结合使用。

### 3. 反证法

假设贪心解不是最优解，推导出矛盾。

### 4. 拟阵（Matroid）理论

对于某些组合优化问题，如果其可行解集构成一个拟阵，则贪心法一定可以求得最优解。最小生成树（Kruskal 算法）就是拟阵贪心的典型应用。该方法偏理论，实际刷题中较少直接使用。

---

## 五、经典例题

### 例题 1：分发饼干（LeetCode 455）

**问题**：每个孩子有胃口值 `g[i]`，每块饼干有尺寸 `s[j]`。只有饼干尺寸大于等于孩子胃口时才能满足。求最多能满足多少个孩子。

**贪心策略**：将孩子和饼干都升序排序，用最小的饼干去满足最容易满足的孩子。

```cpp
int findContentChildren(vector<int>& g, vector<int>& s) {
    sort(g.begin(), g.end());
    sort(s.begin(), s.end());
    int i = 0, j = 0;
    while (i < (int)g.size() && j < (int)s.size()) {
        if (s[j] >= g[i]) ++i;
        ++j;
    }
    return i;
}
```

**复杂度**：$O(n \log n + m \log m)$

**证明要点**：优先满足胃口最小的孩子，不会浪费大饼干，交换论证可证。

---

### 例题 2：无重叠区间（LeetCode 435）

**问题**：给定一组区间，求最少删除多少个区间，使得剩余区间互不重叠。

**贪心策略**：按区间右端点升序排序，每次选择右端点最小且与已选区间不重叠的区间。等价于求最多能保留多少个不重叠区间。

```cpp
int eraseOverlapIntervals(vector<vector<int>>& intervals) {
    if (intervals.empty()) return 0;
    sort(intervals.begin(), intervals.end(),
         [](const auto& a, const auto& b) { return a[1] < b[1]; });

    int count = 1;
    int end = intervals[0][1];
    for (int i = 1; i < (int)intervals.size(); ++i) {
        if (intervals[i][0] >= end) {
            ++count;
            end = intervals[i][1];
        }
    }
    return (int)intervals.size() - count;
}
```

**复杂度**：$O(n \log n)$

**证明要点**：与活动选择问题相同，选择右端点最小的区间可以为后续留下最多空间。

---

### 例题 3：用最少数量的箭引爆气球（LeetCode 452）

**问题**：气球在水平方向上占据区间 `[x_start, x_end]`，一支箭可以引爆所有与射箭位置相交的气球。求最少箭数。

**贪心策略**：按右端点升序排序，每次在第一个未引爆气球的右端点射箭，然后跳过所有被引爆的气球。

```cpp
int findMinArrowShots(vector<vector<int>>& points) {
    if (points.empty()) return 0;
    sort(points.begin(), points.end(),
         [](const auto& a, const auto& b) { return a[1] < b[1]; });

    int arrows = 1;
    int end = points[0][1];
    for (int i = 1; i < (int)points.size(); ++i) {
        if (points[i][0] > end) {
            ++arrows;
            end = points[i][1];
        }
    }
    return arrows;
}
```

**复杂度**：$O(n \log n)$

---

### 例题 4：跳跃游戏（LeetCode 55）

**问题**：给定非负整数数组 `nums`，你最初位于第一个下标，每个元素表示你在该位置可以跳跃的最大长度。判断能否到达最后一个下标。

**贪心策略**：维护当前能到达的最远位置 `maxReach`，遍历数组并更新。

```cpp
bool canJump(vector<int>& nums) {
    int maxReach = 0;
    for (int i = 0; i < (int)nums.size(); ++i) {
        if (i > maxReach) return false;
        maxReach = max(maxReach, i + nums[i]);
    }
    return true;
}
```

**复杂度**：$O(n)$

---

### 例题 5：跳跃游戏 II（LeetCode 45）

**问题**：求到达最后一个下标的最少跳跃次数。

**贪心策略**：在每一跳的范围内，选择能到达最远位置的下标作为下一跳的起点。类似 BFS 分层。

```cpp
int jump(vector<int>& nums) {
    int n = nums.size();
    int jumps = 0, curEnd = 0, farthest = 0;
    for (int i = 0; i < n - 1; ++i) {
        farthest = max(farthest, i + nums[i]);
        if (i == curEnd) {
            ++jumps;
            curEnd = farthest;
        }
    }
    return jumps;
}
```

**复杂度**：$O(n)$

---

### 例题 6：加油站（LeetCode 134）

**问题**：环形路线上有 `n` 个加油站，第 `i` 个有 `gas[i]` 升油，到下一个站需要 `cost[i]` 升油。求能绕环路一周的起始站编号，不存在返回 -1。

**贪心策略**：

- 如果总油量小于总消耗，一定无解。
- 否则，从某个站出发，若累计油量变为负数，则说明从该站到当前站之间的任何站出发都无法到达，直接从下一个站重新开始。

```cpp
int canCompleteCircuit(vector<int>& gas, vector<int>& cost) {
    int total = 0, cur = 0, start = 0;
    for (int i = 0; i < (int)gas.size(); ++i) {
        int diff = gas[i] - cost[i];
        total += diff;
        cur += diff;
        if (cur < 0) {
            start = i + 1;
            cur = 0;
        }
    }
    return total >= 0 ? start : -1;
}
```

**复杂度**：$O(n)$

**证明要点**：若从 `i` 出发无法到达 `j`，则从 `i` 到 `j` 之间任意站出发也无法到达 `j`，因为中间站的剩余油量非负，出发时只会更少。

---

### 例题 7：柠檬水找零（LeetCode 860）

**问题**：每杯柠檬水 5 美元，顾客可能付 5、10、20 美元，你需要正确找零。判断能否完成所有交易。

**贪心策略**：优先使用 10 美元找零，尽量保留 5 美元，因为 5 美元更灵活。

```cpp
bool lemonadeChange(vector<int>& bills) {
    int five = 0, ten = 0;
    for (int b : bills) {
        if (b == 5) {
            ++five;
        } else if (b == 10) {
            if (five == 0) return false;
            --five;
            ++ten;
        } else {  // b == 20
            if (ten > 0 && five > 0) {
                --ten;
                --five;
            } else if (five >= 3) {
                five -= 3;
            } else {
                return false;
            }
        }
    }
    return true;
}
```

**复杂度**：$O(n)$

---

### 例题 8：分数背包

**问题**：物品可分割，背包容量为 `C`，第 `i` 件物品重量 `w[i]`，价值 `v[i]`，求最大价值。

**贪心策略**：按单位重量价值 `v[i]/w[i]` 降序排序，依次装入，最后一件可分割。

```cpp
#include <vector>
#include <algorithm>
using namespace std;

double fractionalKnapsack(vector<int>& w, vector<int>& v, int C) {
    int n = w.size();
    vector<int> idx(n);
    iota(idx.begin(), idx.end(), 0);
    sort(idx.begin(), idx.end(), [&](int a, int b) {
        return (double)v[a] / w[a] > (double)v[b] / w[b];
    });

    double total = 0.0;
    int remain = C;
    for (int i : idx) {
        if (remain >= w[i]) {
            total += v[i];
            remain -= w[i];
        } else {
            total += (double)v[i] * remain / w[i];
            break;
        }
    }
    return total;
}
```

**复杂度**：$O(n \log n)$

**注意**：0-1 背包不能用此贪心策略。

---

### 例题 9：霍夫曼编码

**问题**：给定字符频率，构造最优前缀编码，使编码后总长度最小。

**贪心策略**：每次从优先队列中取出频率最小的两个节点，合并为新节点，频率为两者之和，放回队列。重复直到只剩一个节点。

```cpp
#include <queue>
#include <vector>
using namespace std;

long long huffman(vector<long long>& freq) {
    priority_queue<long long, vector<long long>, greater<long long>> pq(
        freq.begin(), freq.end());

    long long total = 0;
    while (pq.size() > 1) {
        long long a = pq.top(); pq.pop();
        long long b = pq.top(); pq.pop();
        total += a + b;
        pq.push(a + b);
    }
    return total;  // 编码总长度（加权路径长度）
}
```

**复杂度**：$O(n \log n)$

**证明要点**：频率最小的两个字符一定位于最深层，且可以互为兄弟节点，通过交换论证可证。

---

### 例题 10：最小生成树（Kruskal）

**Kruskal 算法**：按边权升序排序，依次选择不构成环的边，直到选够 `n-1` 条边。使用并查集判环。

```cpp
#include <vector>
#include <algorithm>
#include <numeric>
using namespace std;

struct DSU {
    vector<int> parent;
    explicit DSU(int n) : parent(n) {
        iota(parent.begin(), parent.end(), 0);
    }
    int find(int x) {
        return parent[x] == x ? x : parent[x] = find(parent[x]);
    }
    bool unite(int x, int y) {
        int rx = find(x), ry = find(y);
        if (rx == ry) return false;
        parent[rx] = ry;
        return true;
    }
};

struct Edge {
    int u, v, w;
};

long long kruskal(int n, vector<Edge>& edges) {
    sort(edges.begin(), edges.end(),
         [](const Edge& a, const Edge& b) { return a.w < b.w; });

    DSU dsu(n);
    long long mstWeight = 0;
    int count = 0;
    for (const auto& e : edges) {
        if (dsu.unite(e.u, e.v)) {
            mstWeight += e.w;
            if (++count == n - 1) break;
        }
    }
    return mstWeight;
}
```

**复杂度**：$O(E \log E)$

**证明要点**：拟阵理论或切割性质（Cut Property）。

---

## 六、常见陷阱

1. **贪心策略错误**  
   最经典的例子是硬币找零：面额为 `[1, 5, 11]`，目标金额 15。贪心会选择 `11 + 1 + 1 + 1 + 1`（5 枚），而最优解是 `5 + 5 + 5`（3 枚）。贪心失效。

2. **忽略排序**  
   很多贪心问题需要先按某个关键字排序，排序依据选错会导致结果错误。

3. **未证明就使用**  
   竞赛中常见错误：凭直觉认为贪心正确，结果被反例击穿。务必用交换论证或小规模暴力对拍验证。

4. **0-1 背包误用贪心**  
   0-1 背包物品不可分割，按性价比贪心可能留下无法利用的空间，必须用动态规划。

5. **边界条件处理不当**  
   如区间问题中的开闭区间、空输入、单个元素等。

---

## 七、刷题路线

**入门（建立直觉）**

- [ ] LeetCode 455. 分发饼干
- [ ] LeetCode 860. 柠檬水找零
- [ ] LeetCode 55. 跳跃游戏
- [ ] LeetCode 1005. K 次取反后最大化数组和

**进阶（区间与排序贪心）**

- [ ] LeetCode 435. 无重叠区间
- [ ] LeetCode 452. 用最少数量的箭引爆气球
- [ ] LeetCode 45. 跳跃游戏 II
- [ ] LeetCode 134. 加油站
- [ ] LeetCode 376. 摆动序列
- [ ] LeetCode 738. 单调递增的数字

**高阶（经典模型）**

- [ ] LeetCode 122. 买卖股票的最佳时机 II
- [ ] LeetCode 406. 根据身高重建队列
- [ ] LeetCode 621. 任务调度器
- [ ] 霍夫曼编码
- [ ] 最小生成树（Kruskal / Prim）
- [ ] 单源最短路（Dijkstra）

---

## 八、总结

贪心算法是一种强大而优雅的算法思想，但它的适用条件比动态规划更苛刻。掌握贪心法的关键在于：

1. **识别贪心选择性质**：局部最优能否推出全局最优？
2. **设计贪心策略**：按什么关键字排序？每次选择什么？
3. **证明正确性**：交换论证、归纳法、反证法。
4. **警惕反例**：不确定时，先写暴力对拍。

一句话概括贪心法：

> **每一步都选当前最好的，且永不后悔——但前提是，你能证明这样做不会错过全局最优。**

---

_本文基于 LeetCode 题库与经典算法教材整理，代码示例使用 C++17。_
