---
title: "算法学习手册 02：常用数据结构与搜索"
description: "哈希、栈、单调栈、队列、单调队列、堆，以及 DFS、BFS 和回溯。"
date: 2026-08-25
category: "Algorithms"
tags: ["Algorithms", "Data Structures", "DFS", "BFS", "C++", "Python"]
draft: false
---

[← 返回算法手册目录](/notes/algorithm-handbook-index)

## 1. 哈希、map 与 set

哈希表适合频次统计、存在性查询、去重和寻找 complement。有序 `map/set` 则额外维护顺序。

```cpp
unordered_map<long long, int> cnt;
for (auto x : a) cnt[x]++;
unordered_set<int> seen;
```

```python
from collections import Counter
cnt = Counter(a)
seen = set(a)
```

**坑点：**`map` 的 `[]` 会创建键；需要有序时不要随手使用 `unordered_map`。

## 2. 栈与单调栈

普通栈维护“最近的未处理对象”。单调栈进一步维持栈内单调性，使已经被淘汰的元素以后也不再需要。

典型信号：左/右第一个更大或更小、柱状图、贡献法。

```cpp
stack<int> st;
for (int i = 0; i < n; i++) {
    while (!st.empty() && a[st.top()] <= a[i]) st.pop();
    // st.top() 是左侧最近更大候选
    st.push(i);
}
```

```python
st = []
for i, x in enumerate(a):
    while st and a[st[-1]] <= x:
        st.pop()
    st.append(i)
```

栈里通常存**下标**，因为下标既能找到值，也能计算距离和判断位置。

## 3. 队列、deque 与单调队列

队列是 FIFO；单调队列常用来在线维护滑动窗口最大值或最小值。

窗口右移时：队头删除过期元素，队尾淘汰不可能再成为最优值的候选。

```cpp
deque<int> q;
for (int i = 0; i < n; i++) {
    while (!q.empty() && q.front() <= i - k) q.pop_front();
    while (!q.empty() && a[q.back()] <= a[i]) q.pop_back();
    q.push_back(i);
    if (i >= k - 1) ans.push_back(a[q.front()]);
}
```

## 4. 堆 / priority_queue

堆不保证整体有序，只保证堆顶是当前最大或最小元素，因此适合：Top K、反复取最值、合并代价、Dijkstra。

```cpp
priority_queue<int> mx;
priority_queue<int, vector<int>, greater<int>> mn;
```

```python
import heapq
pq = []
heapq.heappush(pq, x)
x = heapq.heappop(pq)
```

C++ 默认大根堆，Python `heapq` 默认小根堆。

## 5. DFS：一路走到底再回来

DFS 常用于连通块、树遍历、可达性和搜索全部状态。

```cpp
void dfs(int u) {
    vis[u] = 1;
    for (int v : g[u])
        if (!vis[v]) dfs(v);
}
```

```python
def dfs(u):
    vis[u] = True
    for v in g[u]:
        if not vis[v]:
            dfs(v)
```

图上 DFS 一定注意 `visited`，否则可能在环中无限递归。

## 6. BFS：按层扩散

BFS 一层层向外扩散。在无权图中，从起点第一次到达某点时，经过的边数就是最少的。

```cpp
queue<int> q;
q.push(s);
dist[s] = 0;
while (!q.empty()) {
    int u = q.front(); q.pop();
    for (int v : g[u]) if (dist[v] == -1) {
        dist[v] = dist[u] + 1;
        q.push(v);
    }
}
```

```python
from collections import deque
q = deque([s])
dist[s] = 0
while q:
    u = q.popleft()
    for v in g[u]:
        if dist[v] == -1:
            dist[v] = dist[u] + 1
            q.append(v)
```

**坑点：**通常在“入队时”就标记，否则同一节点可能重复入队。

## 7. 回溯：选择 → 递归 → 撤销

回溯是 DFS 的一种典型形式，常见于排列、组合、子集和棋盘搜索。

```cpp
void dfs() {
    if (path.size() == n) {
        // 记录答案
        return;
    }
    for (int i = 1; i <= n; i++) if (!used[i]) {
        used[i] = 1;
        path.push_back(i);
        dfs();
        path.pop_back();
        used[i] = 0;
    }
}
```

```python
def dfs():
    if len(path) == n:
        ans.append(path[:])
        return
    for i in range(1, n + 1):
        if not used[i]:
            used[i] = True
            path.append(i)
            dfs()
            path.pop()
            used[i] = False
```

Python 保存路径时注意 `path[:]`，否则保存的是同一个可变列表引用。

## 练习

- LeetCode 1：Two Sum
- LeetCode 84：Largest Rectangle in Histogram
- LeetCode 239：Sliding Window Maximum
- LeetCode 215：Kth Largest Element
- LeetCode 200：Number of Islands
- LeetCode 994：Rotting Oranges
- LeetCode 46：Permutations
- LeetCode 78：Subsets

[← 第一章](/notes/algorithm-handbook-01-foundations-array) · [第三章：动态规划 →](/notes/algorithm-handbook-03-dynamic-programming)
