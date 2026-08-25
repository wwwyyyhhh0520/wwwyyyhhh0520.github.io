---
title: "算法学习手册 04：图论"
description: "并查集、拓扑排序、Dijkstra 与 Kruskal 最小生成树。"
date: 2026-08-25
category: "Algorithms"
tags: ["Algorithms", "Graph", "DSU", "Dijkstra", "MST", "C++", "Python"]
draft: false
---

[← 返回算法手册目录](/notes/algorithm-handbook-index)

DFS/BFS 已经放在第二章。这一章继续整理图论里几种常见问题：动态连通、依赖关系、带权最短路和最小生成树。

## 1. 并查集 DSU

并查集不关心两个点具体通过哪条路径相连，只关心它们是否属于同一个集合。

核心操作：`find` 找代表元，`union` 合并集合。

```cpp
vector<int> fa(n + 1);
iota(fa.begin(), fa.end(), 0);

function<int(int)> find = [&](int x) {
    return fa[x] == x ? x : fa[x] = find(fa[x]);
};
```

```python
fa = list(range(n + 1))

def find(x):
    if fa[x] != x:
        fa[x] = find(fa[x])
    return fa[x]

def unite(a, b):
    a, b = find(a), find(b)
    if a != b:
        fa[a] = b
```

典型问题：动态连通、朋友/帮派、加边判环、Kruskal。

## 2. 拓扑排序

拓扑排序处理 DAG 中的依赖关系。基本做法是不断取入度为 0 的节点，然后删除它的出边。

```cpp
queue<int> q;
for (int i = 1; i <= n; i++)
    if (indeg[i] == 0) q.push(i);

int cnt = 0;
while (!q.empty()) {
    int u = q.front(); q.pop();
    cnt++;
    for (int v : g[u])
        if (--indeg[v] == 0) q.push(v);
}

bool acyclic = (cnt == n);
```

如果最终处理的节点数量少于 `n`，说明图中存在环。

## 3. Dijkstra 最短路

Dijkstra 用于**非负边权**的单源最短路。

核心思路：优先队列每次取当前距离最小的状态，然后尝试松弛它的邻边。

```cpp
vector<long long> d(n + 1, INF);
d[s] = 0;
priority_queue<pair<ll,int>, vector<pair<ll,int>>, greater<pair<ll,int>>> pq;
pq.push({0, s});

while (!pq.empty()) {
    auto [du, u] = pq.top();
    pq.pop();
    if (du != d[u]) continue;

    for (auto [v, w] : g[u]) {
        if (d[v] > du + w) {
            d[v] = du + w;
            pq.push({d[v], v});
        }
    }
}
```

```python
import heapq

d = [10**30] * n
d[s] = 0
pq = [(0, s)]

while pq:
    du, u = heapq.heappop(pq)
    if du != d[u]:
        continue
    for v, w in g[u]:
        nd = du + w
        if nd < d[v]:
            d[v] = nd
            heapq.heappush(pq, (nd, v))
```

**注意：存在负权边时不能直接使用普通 Dijkstra。**

## 4. Kruskal 最小生成树

目标是让无向连通图中的所有点连通，并使选择的边权总和最小。

Kruskal 的策略很直接：所有边按权值从小到大排序，只要加入当前边不会形成环，就选择它。

```cpp
sort(edges.begin(), edges.end());
for (auto [w, u, v] : edges) {
    if (find(u) != find(v)) {
        unite(u, v);
        ans += w;
        used++;
    }
}
```

判环正好可以交给前面的 DSU，因此这两个知识点通常一起学习。

## 识别问题时的简单分类

```text
无权最短路      → BFS
非负带权最短路  → Dijkstra
动态判断连通性  → DSU
任务/课程依赖   → Topological Sort
连接全部点且代价最小 → MST / Kruskal
```

## 练习

- LeetCode 684：Redundant Connection
- LeetCode 207：Course Schedule
- LeetCode 743：Network Delay Time
- LeetCode 1584：Min Cost to Connect All Points
- 洛谷 P3367：并查集
- 洛谷 P4779：单源最短路径
- 洛谷 P3366：最小生成树

[← 第三章](/notes/algorithm-handbook-03-dynamic-programming) · [第五章：数学、位运算与字符串 →](/notes/algorithm-handbook-05-math-bit-string)
