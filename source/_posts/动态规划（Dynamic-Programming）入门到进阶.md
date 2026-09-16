---
title: 动态规划（Dynamic Programming）入门到进阶
date: 2026-09-16 15:45:32
tags:
  - Dynamic Programming
  - 动态规划
categories: Algorithms
---

> 动态规划不是一种具体的算法，而是一种**思考问题的方式**。它的核心思想是：**把大问题拆成小问题，把重复计算的结果存起来，用空间换时间。**

---

## 一、先讲个故事

斐波那契数列大家都会写：

> 本文示例统一用 C++ 编写，默认已经 `#include <bits/stdc++.h>` 并 `using namespace std;`。

```cpp
int fib(int n) {
    if (n <= 1) return n;
    return fib(n - 1) + fib(n - 2);
}
```

这段代码的递归树长这样（以 `fib(5)` 为例）：

```
                fib(5)
              /        \
          fib(4)       fib(3)
         /     \       /     \
      fib(3) fib(2) fib(2) fib(1)
      /   \
   fib(2) fib(1)
```

`fib(3)` 被算了 2 次，`fib(2)` 被算了 3 次。当 `n = 50` 时，程序会直接卡死——时间复杂度是 $O(2^n)$。

问题出在哪？**重复子问题**。

如果第一次算出 `fib(3)` 时，就把它记在小本本上，下次直接查表，那复杂度立刻降到 $O(n)$。这就是动态规划最朴素的形态：**记忆化搜索**。

```cpp
const int MAXN = 100;
int memo[MAXN];          // 初始化为 -1，表示「还没算过」

int fib(int n) {
    if (n <= 1) return n;
    if (memo[n] != -1) return memo[n];
    return memo[n] = fib(n - 1) + fib(n - 2);
}

// 调用前先 memset(memo, -1, sizeof(memo));
```

---

## 二、什么时候能用动态规划？

一道题能用 DP 解，必须同时满足两个条件：

| 条件           | 含义                                         | 判断方法                                         |
| -------------- | -------------------------------------------- | ------------------------------------------------ |
| **最优子结构** | 大问题的最优解，可以由小问题的最优解推导出来 | 问自己：知道了子问题答案，能不能推出原问题答案？ |
| **重叠子问题** | 不同的决策路径会重复计算同一批子问题         | 画递归树，看有没有重复节点                       |

另外还有一个隐含前提：**无后效性**——当前状态一旦确定，后续决策只依赖当前状态，不关心这个状态是怎么来的。

> 💡 **贪心 vs 动规 vs 分治**
>
> - 分治：子问题**互相独立**，不重叠（如归并排序）
> - 贪心：每步都取局部最优，且**能证明**局部最优 = 全局最优
> - 动规：子问题重叠，需要**记录并比较**所有可能的选择

---

## 三、动态规划解题五步法

这是本文最重要的部分，建议背下来。

### 1️⃣ 确定状态（State）

状态就是「我要记录什么」。通常定义为：

```
dp[i]      = 前 i 个元素的最优解
dp[i][j]   = 从 i 到 j 的区间最优解
dp[i][j]   = 第一个序列前 i 个、第二个序列前 j 个的最优解
```

**关键点**：状态必须能完整描述当前局面，并且足够「小」。

### 2️⃣ 推导状态转移方程（Transition）

这是最难的一步。问自己：**dp[i] 可以由哪些更小的状态推出来？**

```
dp[i] = max(dp[i-1], dp[i-2] + nums[i])
```

写不出方程，通常是因为**状态定义得不好**。回头改第 1 步。

### 3️⃣ 初始化（Base Case）

最小的、不能再拆的状态是什么？它们的值是多少？

```cpp
dp[0] = 0;
dp[1] = 1;
```

### 4️⃣ 确定遍历顺序

必须保证：**计算 dp[i] 时，它依赖的状态已经算好了**。

- 一维线性 DP：`i` 从小到大
- 二维区间 DP：按区间长度从小到大
- 背包问题：注意正序 / 逆序（后面详细讲）

### 5️⃣ 确定答案位置

是 `dp[n]`？`max(dp)`？还是 `dp[0][n-1]`？

> ⚠️ 90% 的 DP 写错，都错在第 1 步（状态定义）或第 4 步（遍历顺序）。

---

## 四、经典例题精讲

### 例题 1：爬楼梯（入门）

> 每次可以爬 1 或 2 个台阶，爬到第 n 阶有多少种方法？

**状态**：`dp[i]` = 爬到第 i 阶的方法数
**转移**：最后一步要么从 i-1 走上来，要么从 i-2 跳上来

$$dp[i] = dp[i-1] + dp[i-2]$$

**初始化**：`dp[0] = 1`（原地不动算一种），`dp[1] = 1`

```cpp
int climbStairs(int n) {
    if (n <= 1) return 1;
    int prev = 1, curr = 1;
    for (int i = 2; i <= n; ++i) {
        int next = prev + curr;
        prev = curr;
        curr = next;
    }
    return curr;
}
```

这里只用了两个变量，因为 `dp[i]` 只依赖前两项——这就是**滚动数组**思想的雏形。

---

### 例题 2：打家劫舍（线性 DP）

> 一排房子，不能偷相邻的两家，求最大金额。

**状态**：`dp[i]` = 考虑前 i 家能偷到的最大金额
**转移**：

- 不偷第 i 家：`dp[i-1]`
- 偷第 i 家：`dp[i-2] + nums[i]`

$$dp[i] = \max(dp[i-1],\ dp[i-2] + nums[i])$$

```cpp
int rob(vector<int>& nums) {
    int prev = 0, curr = 0;   // prev = dp[i-2], curr = dp[i-1]
    for (int num : nums) {
        int next = max(curr, prev + num);
        prev = curr;
        curr = next;
    }
    return curr;
}
```

---

### 例题 3：0-1 背包（必考）

> 有 n 件物品，第 i 件重 `w[i]`、价值 `v[i]`，背包容量为 C。每件物品只能取 0 次或 1 次，求最大价值。

**状态**：`dp[i][j]` = 只考虑前 i 件物品、容量为 j 时的最大价值
**转移**：

- 不拿第 i 件：`dp[i-1][j]`
- 拿第 i 件（前提 `j >= w[i]`）：`dp[i-1][j - w[i]] + v[i]`

$$dp[i][j] = \max\Big(dp[i-1][j],\ dp[i-1][j-w_i] + v_i\Big)$$

填表过程（容量从 0 到 C 逐步扩张）：

| i \ j         | 0   | 1   | 2   | 3   | 4   |
| ------------- | --- | --- | --- | --- | --- |
| 0（无物品）   | 0   | 0   | 0   | 0   | 0   |
| 1（w=1,v=15） | 0   | 15  | 15  | 15  | 15  |
| 2（w=3,v=20） | 0   | 15  | 15  | 20  | 35  |
| 3（w=4,v=30） | 0   | 15  | 15  | 20  | 35  |

**二维写法**：

```cpp
int knapsack(const vector<int>& w, const vector<int>& v, int C) {
    int n = w.size();
    vector<vector<int>> dp(n + 1, vector<int>(C + 1, 0));
    for (int i = 1; i <= n; ++i) {
        for (int j = 0; j <= C; ++j) {
            dp[i][j] = dp[i - 1][j];
            if (j >= w[i - 1]) {
                dp[i][j] = max(dp[i][j], dp[i - 1][j - w[i - 1]] + v[i - 1]);
            }
        }
    }
    return dp[n][C];
}
```

**一维优化（重点）**：

观察发现，`dp[i][*]` 只依赖 `dp[i-1][*]`，所以可以把第一维压掉。但**容量必须逆序遍历**：

```cpp
int knapsack(const vector<int>& w, const vector<int>& v, int C) {
    vector<int> dp(C + 1, 0);
    for (int i = 0; i < (int)w.size(); ++i) {
        for (int j = C; j >= w[i]; --j) {   // ← 逆序！
            dp[j] = max(dp[j], dp[j - w[i]] + v[i]);
        }
    }
    return dp[C];
}
```

> 🤔 **为什么必须逆序？**
>
> 因为 `dp[j]` 依赖的是**上一轮**的 `dp[j - w[i]]`。如果正序遍历，`dp[j - w[i]]` 已经在**本轮**被更新过了，相当于同一件物品被拿了多次——那就变成了**完全背包**。
>
> 一句话记忆：**0-1 背包逆序，完全背包正序。**

**完全背包**（物品可以无限取）：

```cpp
for (int i = 0; i < (int)w.size(); ++i) {
    for (int j = w[i]; j <= C; ++j) {   // ← 正序
        dp[j] = max(dp[j], dp[j - w[i]] + v[i]);
    }
}
```

---

### 例题 4：最长递增子序列 LIS

> 求数组中最长严格递增子序列的长度。

**状态**：`dp[i]` = 以 `nums[i]` **结尾**的最长递增子序列长度

> ⚠️ 注意状态定义里的「以 i 结尾」，这是 LIS 的关键。如果定义成「前 i 个元素中的 LIS」，就无法转移了。

**转移**：枚举所有 `j < i` 且 `nums[j] < nums[i]`

$$dp[i] = \max_{j<i,\ nums[j]<nums[i]}\big(dp[j]\big) + 1$$

```cpp
int lengthOfLIS(const vector<int>& nums) {
    if (nums.empty()) return 0;
    vector<int> dp(nums.size(), 1);
    for (int i = 0; i < (int)nums.size(); ++i) {
        for (int j = 0; j < i; ++j) {
            if (nums[j] < nums[i]) {
                dp[i] = max(dp[i], dp[j] + 1);
            }
        }
    }
    return *max_element(dp.begin(), dp.end());   // 答案不是 dp.back()，而是全局最大值
}
```

时间复杂度 $O(n^2)$。用贪心 + 二分可以优化到 $O(n\log n)$。

---

### 例题 5：最长公共子序列 LCS

> 求两个字符串的最长公共子序列长度（不要求连续）。

**状态**：`dp[i][j]` = `text1` 前 i 个字符与 `text2` 前 j 个字符的 LCS 长度

**转移**：

$$
dp[i][j] =
\begin{cases}
dp[i-1][j-1] + 1, & \text{若 } text1[i-1] = text2[j-1] \\[4pt]
\max\big(dp[i-1][j],\ dp[i][j-1]\big), & \text{否则}
\end{cases}
$$

```cpp
int longestCommonSubsequence(const string& text1, const string& text2) {
    int m = text1.size(), n = text2.size();
    vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));
    for (int i = 1; i <= m; ++i) {
        for (int j = 1; j <= n; ++j) {
            if (text1[i - 1] == text2[j - 1]) {
                dp[i][j] = dp[i - 1][j - 1] + 1;
            } else {
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
    }
    return dp[m][n];
}
```

注意：多开一行一列作为**哨兵**，表示「空串」，可以省去大量边界判断。这是二维 DP 的通用技巧。

---

### 例题 6：编辑距离

> 把 `word1` 变成 `word2`，允许插入、删除、替换，求最少操作次数。

**状态**：`dp[i][j]` = `word1` 前 i 个字符转成 `word2` 前 j 个字符的最少操作数

**转移**：

- 字符相同：`dp[i-1][j-1]`
- 字符不同：`min(替换, 删除, 插入) + 1`

```cpp
int minDistance(const string& word1, const string& word2) {
    int m = word1.size(), n = word2.size();
    vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));
    for (int i = 0; i <= m; ++i) dp[i][0] = i;   // 全删
    for (int j = 0; j <= n; ++j) dp[0][j] = j;   // 全插
    for (int i = 1; i <= m; ++i) {
        for (int j = 1; j <= n; ++j) {
            if (word1[i - 1] == word2[j - 1]) {
                dp[i][j] = dp[i - 1][j - 1];
            } else {
                dp[i][j] = min({dp[i - 1][j - 1],    // 替换
                                dp[i - 1][j],        // 删除
                                dp[i][j - 1]}) + 1;  // 插入
            }
        }
    }
    return dp[m][n];
}
```

---

### 例题 7：零钱兑换

> 硬币面额 `coins`，凑出金额 `amount` 所需的最少硬币数，凑不出返回 -1。

**状态**：`dp[j]` = 凑出金额 j 的最少硬币数
**转移**：$dp[j] = \min(dp[j],\ dp[j - coin] + 1)$

```cpp
int coinChange(const vector<int>& coins, int amount) {
    const int INF = 1e9;              // 足够大，且 +1 不会溢出
    vector<int> dp(amount + 1, INF);
    dp[0] = 0;
    for (int coin : coins) {          // 物品在外
        for (int j = coin; j <= amount; ++j) {
            dp[j] = min(dp[j], dp[j - coin] + 1);
        }
    }
    return dp[amount] == INF ? -1 : dp[amount];
}
```

> 这是**完全背包**的变体（每种硬币无限个），所以内层正序。

---

## 五、空间优化：滚动数组

DP 的空间往往可以压缩。规律如下：

| 转移依赖                            | 优化方法                      | 示例             |
| ----------------------------------- | ----------------------------- | ---------------- |
| 只依赖 `dp[i-1]`                    | 两个变量 / 一维数组           | 爬楼梯、打家劫舍 |
| 依赖 `dp[i-1][*]` 整行              | 一维数组 + 逆序               | 0-1 背包         |
| 依赖 `dp[i-1][j-1]` 和 `dp[i-1][j]` | 一维数组 + 临时变量保存左上角 | LCS、编辑距离    |

以编辑距离为例，二维压一维需要额外记录被覆盖的 `dp[i-1][j-1]`：

```cpp
int minDistance(const string& word1, const string& word2) {
    int m = word1.size(), n = word2.size();
    vector<int> dp(n + 1);
    for (int j = 0; j <= n; ++j) dp[j] = j;
    for (int i = 1; i <= m; ++i) {
        int prev = dp[0];   // 保存 dp[i-1][j-1]
        dp[0] = i;
        for (int j = 1; j <= n; ++j) {
            int temp = dp[j];
            if (word1[i - 1] == word2[j - 1]) {
                dp[j] = prev;
            } else {
                dp[j] = min({prev, dp[j], dp[j - 1]}) + 1;
            }
            prev = temp;
        }
    }
    return dp[n];
}
```

---

## 六、记忆化搜索 vs 递推

两种写法本质等价，选哪个看个人习惯和题目特点。

```cpp
// 写法一：记忆化搜索（自顶向下）
int memo[100];   // 初始化为 -1

int solve(int n) {
    if (n <= 2) return n;
    if (memo[n] != -1) return memo[n];
    return memo[n] = solve(n - 1) + solve(n - 2);
}

// 写法二：递推（自底向上）
int solve(int n) {
    vector<int> dp(n + 1);
    dp[0] = dp[1] = 1;
    for (int i = 2; i <= n; ++i) {
        dp[i] = dp[i - 1] + dp[i - 2];
    }
    return dp[n];
}
```

|      | 记忆化搜索                 | 递推                     |
| ---- | -------------------------- | ------------------------ |
| 方向 | 自顶向下                   | 自底向上                 |
| 优点 | 只计算需要的状态，思路直观 | 无递归开销，便于空间优化 |
| 缺点 | 递归栈可能溢出             | 可能计算无用状态         |
| 适用 | 状态空间大但可达状态少     | 状态规整、需要卡常数     |

> 💡 **实战建议**：先用记忆化搜索把转移方程写对，再改写成递推优化空间。这样不容易出错。

---

## 七、常见陷阱清单

1. **状态定义不完整** —— 比如 LIS 定义成「前 i 个的 LIS」就错了，必须「以 i 结尾」。
2. **遍历顺序错误** —— 0-1 背包写成正序，等于把物品拿了无数次。
3. **初始化遗漏** —— 求最小值时忘记把 `dp` 初始化成 `INF`，结果全是 0。
4. **答案位置搞错** —— LIS 的答案是 `max(dp)` 而不是 `dp[n-1]`。
5. **数组越界** —— 二维 DP 建议开 `(m+1) × (n+1)`，用哨兵行/列处理边界。
6. **整数溢出** —— 用 `INT_MAX` 当无穷大时，`INT_MAX + 1` 会溢出成负数，最值判断就全错了；建议用 `0x3f3f3f3f` 或 `1e9` 这类「足够大但加了 1 也不会溢出」的值。

---

## 八、DP 的常见分类

| 类型      | 特征                            | 代表题                       |
| --------- | ------------------------------- | ---------------------------- |
| 线性 DP   | 状态沿一维线性推进              | 爬楼梯、打家劫舍、LIS        |
| 区间 DP   | 状态是区间 `[i, j]`，按长度枚举 | 最长回文子串、石子合并       |
| 背包 DP   | 选/不选 + 容量约束              | 0-1 背包、完全背包、多重背包 |
| 双序列 DP | 两个序列的二维表                | LCS、编辑距离                |
| 树形 DP   | 在树上做 DP，常配合后序遍历     | 打家劫舍 III、树的最大独立集 |
| 状压 DP   | 用二进制数表示集合状态          | 旅行商问题、棋盘覆盖         |
| 数位 DP   | 按位统计满足条件的数字个数      | 计数问题                     |

---

## 九、刷题路线推荐

**入门（建立感觉）**

- [ ] LeetCode 70. 爬楼梯
- [ ] LeetCode 198. 打家劫舍
- [ ] LeetCode 53. 最大子数组和
- [ ] LeetCode 509. 斐波那契数

**进阶（掌握套路）**

- [ ] LeetCode 300. 最长递增子序列
- [ ] LeetCode 1143. 最长公共子序列
- [ ] LeetCode 72. 编辑距离
- [ ] LeetCode 322. 零钱兑换
- [ ] LeetCode 416. 分割等和子集

**高阶（挑战）**

- [ ] LeetCode 5. 最长回文子串（区间 DP）
- [ ] LeetCode 337. 打家劫舍 III（树形 DP）
- [ ] LeetCode 312. 戳气球（区间 DP）
- [ ] LeetCode 887. 鸡蛋掉落

---

## 十、总结

动态规划的学习曲线是这样的：

```
看不懂 → 背模板 → 勉强能写 → 偶尔卡壳 → 形成直觉
```

突破瓶颈的关键，不在于刷题数量，而在于每次做题都**老老实实回答五个问题**：

1. `dp` 数组的含义是什么？（用一句话说清楚，最好带上下标含义）
2. 转移方程怎么来的？为什么漏掉某些情况不会出错？
3. 初始值是什么？
4. 循环顺序为什么是这样？
5. 答案是哪个位置？

当你能不假思索地回答这五个问题，DP 就不再是玄学了。

> 「动态规划的本质，是用**有序的枚举**代替**暴力的枚举**，用**记忆化**消除**重复的计算**。」
>
> —— 共勉

---

_如果这篇文章对你有帮助，欢迎点赞收藏。_
