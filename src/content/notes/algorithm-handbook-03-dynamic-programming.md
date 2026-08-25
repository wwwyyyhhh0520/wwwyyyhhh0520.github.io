---
title: "算法学习手册 03：动态规划"
description: "从状态定义开始，整理 0/1 背包、完全背包、LIS 与区间 DP。"
date: 2026-08-25
category: "Algorithms"
tags: ["Algorithms", "Dynamic Programming", "Knapsack", "LIS", "C++", "Python"]
draft: false
---

[← 返回算法手册目录](/notes/algorithm-handbook-index)

## 1. DP 最重要的是先定义状态

DP 的本质是缓存重复子问题。比背公式更重要的是先回答四个问题：

1. `dp[...]` 表示什么？
2. 初值是什么？
3. 状态如何转移？
4. 最终答案在哪里？

例如爬楼梯，定义 `dp[i]` 为到达第 `i` 阶的方案数。最后一步只可能来自 `i-1` 或 `i-2`：

```cpp
vector<long long> dp(n + 1);
dp[0] = 1;
dp[1] = 1;
for (int i = 2; i <= n; i++)
    dp[i] = dp[i - 1] + dp[i - 2];
```

## 2. 0/1 背包

每件物品最多选一次。令 `dp[j]` 表示容量为 `j` 时能获得的最优价值。

一维优化时容量必须**倒序**：

```cpp
vector<long long> dp(W + 1);
for (auto [w, v] : items)
    for (int j = W; j >= w; j--)
        dp[j] = max(dp[j], dp[j - w] + v);
```

```python
dp = [0] * (W + 1)
for w, v in items:
    for j in range(W, w - 1, -1):
        dp[j] = max(dp[j], dp[j - w] + v)
```

倒序的原因是 `dp[j-w]` 仍然来自上一轮，当前物品不会在同一轮被重复使用。

## 3. 完全背包

完全背包中，每种物品可以选无限次。和 0/1 背包最直观的区别是容量**正序**更新：

```cpp
for (auto [w, v] : items)
    for (int j = w; j <= W; j++)
        dp[j] = max(dp[j], dp[j - w] + v);
```

```python
for w, v in items:
    for j in range(w, W + 1):
        dp[j] = max(dp[j], dp[j - w] + v)
```

正序时 `dp[j-w]` 可能已经在当前轮使用过该物品，因此自然允许重复选择。

## 4. 最长上升子序列 LIS

基础 DP 可以定义：`dp[i] = 以 i 结尾的 LIS 长度`，复杂度 O(n²)。

更进一步可以用贪心 + 二分维护 `tails`：`tails[len]` 表示长度为 `len+1` 的上升子序列中，最小可能的结尾。

```cpp
vector<int> tails;
for (int x : a) {
    auto it = lower_bound(tails.begin(), tails.end(), x);
    if (it == tails.end()) tails.push_back(x);
    else *it = x;
}
```

```python
import bisect

tails = []
for x in a:
    p = bisect.bisect_left(tails, x)
    if p == len(tails):
        tails.append(x)
    else:
        tails[p] = x
```

注意：`tails` 本身不一定是原数组中的真实 LIS，它保存的是“更有潜力的结尾状态”。

## 5. 区间 DP

区间 DP 常定义 `dp[l][r]` 为区间 `[l,r]` 的答案。大区间依赖小区间，因此通常按照区间长度从短到长枚举。

```cpp
for (int len = 2; len <= n; len++)
    for (int l = 0; l + len - 1 < n; l++) {
        int r = l + len - 1;
        for (int k = l; k < r; k++)
            dp[l][r] = min(dp[l][r],
                dp[l][k] + dp[k + 1][r] + cost(l, r));
    }
```

典型信号：合并区间、两端决策、回文、区间消除。

## 我现在理解 DP 的方式

看到 DP 题时先不急着写代码，而是先用一句话写出状态定义。如果连 `dp[i]` 的中文含义都说不清楚，后面的转移通常也会很混乱。

然后检查：当前状态到底依赖哪些更小的状态？循环顺序是否保证依赖已经计算完成？不可达状态应该初始化为 0 还是 `INF/-INF`？

## 练习

- LeetCode 70：Climbing Stairs
- LeetCode 53：Maximum Subarray
- LeetCode 416：Partition Equal Subset Sum
- LeetCode 322：Coin Change
- LeetCode 300：Longest Increasing Subsequence
- LeetCode 516：Longest Palindromic Subsequence
- 洛谷 P1048：采药
- 洛谷 P1880：石子合并

[← 第二章](/notes/algorithm-handbook-02-data-structures-search) · [第四章：图论 →](/notes/algorithm-handbook-04-graph)
