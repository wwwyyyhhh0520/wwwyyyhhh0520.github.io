---
title: "算法学习手册 05：数学、位运算与字符串"
description: "GCD/LCM、快速幂、质数筛、状态压缩和 KMP。"
date: 2026-08-25
category: "Algorithms"
tags: ["Algorithms", "Math", "Bitmask", "KMP", "C++", "Python"]
draft: false
---

[← 返回算法手册目录](/notes/algorithm-handbook-index)

## 1. GCD、LCM 与整除

欧几里得算法利用：

```text
gcd(a, b) = gcd(b, a mod b)
```

```cpp
long long g = std::gcd(a, b);
long long l = a / g * b;
```

```python
import math
g = math.gcd(a, b)
l = a // g * b
```

计算 LCM 时先除再乘，可以降低中间结果溢出的风险。

## 2. 快速幂

当指数非常大时，逐次相乘不可行。快速幂利用指数的二进制表示，把复杂度降到 O(log b)。

例如：`13 = 8 + 4 + 1`，因此 `a^13 = a^8 * a^4 * a`。

```cpp
long long qpow(long long a, long long b, long long mod) {
    long long r = 1 % mod;
    while (b) {
        if (b & 1) r = r * a % mod;
        a = a * a % mod;
        b >>= 1;
    }
    return r;
}
```

```python
def qpow(a, b, mod):
    r = 1 % mod
    while b:
        if b & 1:
            r = r * a % mod
        a = a * a % mod
        b >>= 1
    return r

# Python 也可以直接使用：pow(a, b, mod)
```

## 3. 质数、因子与筛法

单个数判素时只需要试除到 `sqrt(n)`；如果需要批量获得 `1..n` 中的质数，可以使用筛法。

```cpp
vector<bool> isp(n + 1, true);
isp[0] = isp[1] = false;
for (int i = 2; i * i <= n; i++) if (isp[i])
    for (long long j = 1LL * i * i; j <= n; j += i)
        isp[j] = false;
```

```python
isp = [True] * (n + 1)
if n >= 0: isp[0] = False
if n >= 1: isp[1] = False

for i in range(2, int(n ** 0.5) + 1):
    if isp[i]:
        for j in range(i * i, n + 1, i):
            isp[j] = False
```

注意 `0` 和 `1` 都不是质数。

## 4. 位运算与状态压缩

整数的每一个二进制位都可以当成一个开关。第 `i` 位为 1 表示集合中选择了第 `i` 个元素。

```cpp
bool has = (mask >> i) & 1;
mask |= (1 << i);       // 加入
mask &= ~(1 << i);      // 删除

for (int mask = 0; mask < (1 << n); mask++) {
    // 枚举所有子集
}
```

```python
has_bit = (mask >> i) & 1
mask |= 1 << i
mask &= ~(1 << i)

for mask in range(1 << n):
    pass
```

状态压缩常见于 `n≈20` 的子集问题，但 O(2^n) 仍然是指数复杂度，不能因为用了 bitmask 就忽略数据范围。

## 5. 字符串与 KMP

KMP 的核心是：模式串失配时，不把已经获得的信息全部丢掉，而是利用已匹配部分的前后缀结构跳转。

前缀函数 `pi[i]` 表示 `s[0..i]` 的最长相等真前后缀长度。

```cpp
vector<int> pi(m);
for (int i = 1; i < m; i++) {
    int j = pi[i - 1];
    while (j && p[i] != p[j]) j = pi[j - 1];
    if (p[i] == p[j]) j++;
    pi[i] = j;
}
```

```python
pi = [0] * len(p)
for i in range(1, len(p)):
    j = pi[i - 1]
    while j and p[i] != p[j]:
        j = pi[j - 1]
    if p[i] == p[j]:
        j += 1
    pi[i] = j
```

学习 KMP 时我更需要理解“为什么能回退到 `pi[j-1]`”，而不是只背一个 `next` 数组模板。

## 练习

- LeetCode 1979：Find GCD of Array
- LeetCode 204：Count Primes
- LeetCode 78：Subsets
- LeetCode 136：Single Number
- LeetCode 28：Find First Occurrence
- 洛谷 P1226：快速幂
- 洛谷 P3383：线性筛素数
- 洛谷 P3375：KMP

[← 第四章](/notes/algorithm-handbook-04-graph) · [第六章：树与进阶数据结构 →](/notes/algorithm-handbook-06-tree-advanced-ds)
