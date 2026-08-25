---
title: "算法学习手册 06：树与进阶数据结构"
description: "树的 DFS / 树形 DP、Fenwick、线段树与 Lazy 思想，以及博弈论基础。"
date: 2026-08-25
category: "Algorithms"
tags: ["Algorithms", "Tree DP", "Fenwick", "Segment Tree", "Game Theory", "C++", "Python"]
draft: false
---

[← 返回算法手册目录](/notes/algorithm-handbook-index)

这一章放一些我准备在基础算法掌握之后继续学习和复习的内容。

## 1. 树的 DFS 与树形 DP

树是连通无环图。DFS 时可以传入 `father` 防止重新走回父节点；树形 DP 通常先计算子树，再把结果合并到父节点。

```cpp
void dfs(int u, int fa) {
    sz[u] = 1;
    for (int v : g[u]) if (v != fa) {
        dfs(v, u);
        sz[u] += sz[v];
    }
}
```

```python
def dfs(u, fa):
    sz[u] = 1
    for v in g[u]:
        if v != fa:
            dfs(v, u)
            sz[u] += sz[v]
```

最简单的例子就是子树大小：

```text
sz[u] = 1 + Σ sz[v]
```

## 2. 树状数组 Fenwick

Fenwick Tree 支持单点修改和前缀和查询，单次复杂度 O(log n)。

关键是：

```text
lowbit(x) = x & -x
```

```cpp
long long tr[N];

void add(int x, long long v) {
    for (; x <= n; x += x & -x)
        tr[x] += v;
}

long long sum(int x) {
    long long r = 0;
    for (; x; x -= x & -x)
        r += tr[x];
    return r;
}
```

```python
def add(x, v):
    while x <= n:
        tr[x] += v
        x += x & -x

def query(x):
    r = 0
    while x:
        r += tr[x]
        x -= x & -x
    return r
```

区间和：`query(r) - query(l-1)`。Fenwick 下标通常从 1 开始。

## 3. 线段树

线段树把 `[1,n]` 不断二分，每个节点维护一段区间的统计信息。

```cpp
void build(int p, int l, int r) {
    if (l == r) {
        tr[p] = a[l];
        return;
    }
    int m = (l + r) / 2;
    build(p * 2, l, m);
    build(p * 2 + 1, m + 1, r);
    tr[p] = tr[p * 2] + tr[p * 2 + 1];
}
```

```python
def build(p, l, r):
    if l == r:
        tr[p] = a[l]
        return
    m = (l + r) // 2
    build(p * 2, l, m)
    build(p * 2 + 1, m + 1, r)
    tr[p] = tr[p * 2] + tr[p * 2 + 1]
```

### Lazy 思想

如果需要整段修改，不必立刻把修改递归到底。可以先把修改“欠”在当前节点的 Lazy 标记里，真正访问子节点时再 `pushdown`。

我准备先熟练 `build / query / 单点 update`，再继续练习 Lazy 区间修改版本。

## 4. 博弈论基础

最通用的入门思路是必胜态和必败态：

- 存在一步可以走到必败态 → 当前是必胜态；
- 所有后继都是必胜态 → 当前是必败态。

Nim 是一个特殊模型。标准 Nim 中，如果所有石堆大小的异或和为 0，则当前玩家处于必败态。

```cpp
long long x = 0;
for (long long a : piles) x ^= a;
cout << (x ? "First" : "Second");
```

```python
x = 0
for a in piles:
    x ^= a
print('First' if x else 'Second')
```

但不能看到“两人轮流操作”就直接套 Nim，首先要确认规则是否真的符合 Nim 模型。

## 练习

- LeetCode 543：Diameter of Binary Tree
- LeetCode 307：Range Sum Query Mutable
- LeetCode 292：Nim Game
- 洛谷 P1352：没有上司的舞会
- 洛谷 P3374：树状数组 1
- 洛谷 P3372：线段树 1
- 洛谷 P2197：Nim 游戏

## 当前学习路线结束

到这里，原始算法笔记中的主要主题就拆完了。后续如果继续学习新的算法，我准备直接按照主题增加新章节，而不是继续把所有内容堆在一个超长 Markdown 文件里。

[← 第五章](/notes/algorithm-handbook-05-math-bit-string) · [回到总目录](/notes/algorithm-handbook-index)
