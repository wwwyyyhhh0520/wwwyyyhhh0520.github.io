---
title: "算法学习手册 01：基础与数组区间"
description: "复杂度、模拟、排序、前缀和、差分、双指针、二分与贪心。"
date: 2026-08-25
category: "Algorithms"
tags: ["Algorithms", "C++", "Python", "Binary Search", "Greedy"]
draft: false
---

[← 返回算法手册目录](/notes/algorithm-handbook-index)

这一章整理我认为最先应该掌握的一组算法思维。很多题并不需要复杂数据结构，先从数据范围、顺序、区间和单调性判断，往往就能找到方向。

## 1. 复杂度分析与数据范围

**优先级：必会｜复杂度：O(1)、O(log n)、O(n)、O(n log n)、O(n²)**

### 算法思想

复杂度不是最后才看的理论，而是写代码前的第一道闸门。先根据 `n` 判断最多能做多少次操作，再决定能不能双重循环，是否需要排序、二分或预处理。

### 识别信号

- 看到 `n`、`q` 的最大值
- 代码出现嵌套循环
- 同一批数据被反复扫描
- 逻辑看起来正确但超时

### 坑点

- 忽略 `while` 套 `for`
- 只看平均情况，不看最坏情况
- C++ 中乘法先溢出再赋给 `long long`，需要 `1LL * a * b`

例如 `n=2e5` 时，O(n²) 大约是 `4e10` 次操作，通常不可行；O(n log n) 往往仍在可接受范围。

## 2. 模拟、枚举与可行性检查

模拟是“题目怎么说就怎么维护状态”；枚举则是在候选不多时逐个假设，再通过 `check()` 验证。

```cpp
bool check(int i) {
    // 假设 i 是答案，验证约束
}
for (int i = 0; i < n; i++)
    if (check(i)) ans.push_back(i);
```

```python
def check(i):
    return True

ans = [i for i in range(n) if check(i)]
```

**常见坑：**把候选枚举写成所有方案枚举，导致指数爆炸；或者状态初始化、数组大小和输入顺序出错。

## 3. 排序：先制造顺序

排序本质上是在制造顺序。顺序出现以后，很多原本需要反复比较的问题就能变成单向扫描。

```cpp
sort(a.begin(), a.end());
sort(v.begin(), v.end(), [](auto &x, auto &y) {
    if (x.first != y.first) return x.first < y.first;
    return x.second < y.second;
});
```

```python
a.sort()
v.sort(key=lambda x: (x[0], x[1]))
```

排序常常是贪心、双指针和二分之前的预处理。

## 4. 前缀和：静态区间查询

如果同一数组需要大量区间求和，可以把重复扫描变成一次预处理。

`pre[i]` 表示前 `i` 个元素的总和，则：

```text
sum(l, r) = pre[r] - pre[l - 1]
```

```cpp
vector<long long> pre(n + 1);
for (int i = 1; i <= n; i++) pre[i] = pre[i - 1] + a[i];
long long sumLR = pre[r] - pre[l - 1];
```

```python
pre = [0] * (n + 1)
for i, x in enumerate(a, 1):
    pre[i] = pre[i - 1] + x
sum_lr = pre[r] - pre[l - 1]
```

**识别信号：**多次静态区间和/计数、`q` 很大、区间信息满足可加减。

## 5. 差分：批量区间修改

差分记录“从这里开始变化多少”。区间 `[l,r]` 加 `x`，只需要：

```text
d[l] += x
d[r + 1] -= x
```

最后做一次前缀和恢复。

```cpp
vector<long long> d(n + 2);
for (auto [l, r, x] : qs) {
    d[l] += x;
    d[r + 1] -= x;
}
for (int i = 1; i <= n; i++) {
    d[i] += d[i - 1];
    a[i] += d[i];
}
```

适合“很多次区间修改，最后统一输出”，但不适合每次修改后立刻在线查询。

## 6. 双指针与滑动窗口

核心是让两个指针尽量只向一个方向移动，使每个元素最多进入和离开窗口常数次。

```cpp
int l = 0;
long long sum = 0;
for (int r = 0; r < n; r++) {
    sum += a[r];
    while (sum > k) sum -= a[l++];
    ans = max(ans, r - l + 1);
}
```

```python
l = 0
s = 0
ans = 0
for r, x in enumerate(a):
    s += x
    while s > k:
        s -= a[l]
        l += 1
    ans = max(ans, r - l + 1)
```

注意：有负数时，“`sum > k` 就移动左端”可能不再具有单调性。

## 7. 二分查找

二分真正依赖的是**单调性**。每一步不是单纯“猜中间”，而是在证明某一半不可能包含答案。

C++：

```cpp
int p = lower_bound(a.begin(), a.end(), x) - a.begin(); // first >= x
int q = upper_bound(a.begin(), a.end(), x) - a.begin(); // first > x
```

Python：

```python
import bisect
p = bisect.bisect_left(a, x)
q = bisect.bisect_right(a, x)
```

### 二分答案

如果不是在数组里找，而是在答案值域里找，可以把候选答案理解成：

```text
不行 不行 不行 | 行 行 行
```

然后寻找分界点。

```cpp
bool check(long long x) { /* x 是否可行 */ }
while (l < r) {
    long long mid = l + (r - l) / 2;
    if (check(mid)) r = mid;
    else l = mid + 1;
}
```

最重要的是先证明 `check(x)` 具有单调性。

## 8. 贪心与交换论证

贪心不是“看起来最优”，而是每一步选择都应该能解释为什么不会让全局最优解变差。

经典思路是交换论证：把某个最优解中的选择替换成我们的贪心选择，如果答案不变差，那么这种选择就是安全的。

例如活动选择问题，可以按结束时间从早到晚排序，优先选择最早结束且不冲突的活动，因为它给后面的活动留下了最多空间。

## 这一章的练习

- LeetCode 303：Range Sum Query
- LeetCode 560：Subarray Sum Equals K
- LeetCode 3：Longest Substring Without Repeating Characters
- LeetCode 704：Binary Search
- LeetCode 875：Koko Eating Bananas
- LeetCode 435：Non-overlapping Intervals
- 洛谷 P2678：跳石头

**下一章：** [常用数据结构与搜索 →](/notes/algorithm-handbook-02-data-structures-search)
