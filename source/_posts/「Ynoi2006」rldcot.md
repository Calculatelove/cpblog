---
title: 「Ynoi2006」rldcot
date: 2025-09-01 23:10:19
updated: 2025-09-01 23:10:19
categories: Ynoi
tags:
  - 启发式合并
  - 扫描线
---

# Description

Link：[Luogu P7880](https://www.luogu.com.cn/problem/P7880)

{% note default %}

给出一棵包含 $n$ 个节点的树，以 $1$ 为根，边带权。

定义 $\mathrm{dep}_x$ 表示节点 $x$ 到根的简单路径上的边权和。

有 $Q$ 次查询，每次查询给出两个参数 $l, r$，对于所有满足 $l \leq i, j \leq r$ 的二元组 $(i, j)$，你需要求出有多少种不同的 $\mathrm{dep}_{\mathrm{LCA}(i, j)}$。

数据范围：$1 \leq n \leq 10^5$，$1 \leq Q \leq 5 \times 10^5$，$-10^9 \leq w \leq 10^9$，$1 \leq l \leq r \leq n$。

时空限制：$500$ms / $512$MiB。

{% endnote %}

<!-- more -->

# Solution

一个简单的想法是：若 $a < b < c < d$ 且 $\mathrm{LCA}(a, d) = \mathrm{LCA}(b, c) = x$，那么 $(b, c)$ 是一定比 $(a, d)$ 更优秀的。

对于树上的一个节点 $u$，我们想要求出一些但不多的二元组 $(i, j)$，满足 $\mathrm{LCA}(i, j) = u$ 并且能覆盖到所有情况（查询时如果漏数了一个二元组，那么一定是有一个更优秀的二元组被数到）。

首先 $i, j$ 必须来自 $u$ 的不同子树，这启发我们在树上进行启发式合并，使用 `std::set` 维护每棵子树的点集。每次合并两棵子树时，我们枚举较小子树的每一个点 $x$，在较大子树的 `std::set` 中查询分别查询 $x$ 的前驱 $\mathrm{pre}$ 后继 $\mathrm{net}$，将 $(\mathrm{pre}, x), (x, \mathrm{suf})$ 加入 $u$ 所拥有的二元组。

这样我们就将二元组个数从 $O(n^2)$ 降到了 $O(n \log n)$！

By the Way：也可以使用 LCT 的 access 操作，求出 $O(n \log n)$ 个二元组。详见 [LOJ 6041](https://www.cnblogs.com/cyx0406/p/11965639.html) 的算法三。

剩余的部分就是很朴素的扫描线了，不过多赘述。

时间复杂度 $\mathcal{O}(n \log^2 n + m \log n)$。

```c++
#include <bits/stdc++.h>

#define debug(a) std::cout << #a << "=" << (a) << ' '

using s64 = long long;
using u64 = unsigned long long;

/* 取 min */
template <class T>
inline void chmin(T &x, const T &y) {
    if (x > y) {
        x = y;
    }
}
/* 取 max */
template <class T>
inline void chmax(T &x, const T &y) {
    if (x < y) {
        x = y;
    }
}

/* ----- ----- ----- 正文 ----- ----- ----- */

const int N = 100100, MaxQ = 500100;

int n, Q;

struct Graph {
    std::vector< std::vector< std::pair<int, int> > > table;

    void init(int _n) {
        table.assign(_n + 1, {});
    }

    void add_edge(int u, int v, int w) {
        table[u].push_back({v, w});
    }
} G;

s64 dist[N];
int id[N];

int len;
s64 mapval[N];

int turn(s64 x) {
    return std::lower_bound(mapval + 1, mapval + 1 + len, x) - mapval;
}

void dfs_init(int u, int fu) {
    for (auto [v, w] : G.table[u]) {
        if (v == fu) continue;
        dist[v] = dist[u] + w;
        dfs_init(v, u);
    }
}

std::set<int> g[N];

std::vector< std::pair<int, int> > scan[N];

std::vector< std::pair<int, int> > qry[N];
int ans[MaxQ];

int lst[N];

namespace BIT {
    int c[N];

    void add(int x, int y) {
        for (; x <= n; x += x & -x) {
            c[x] += y;
        }
    }

    int ask(int x) {
        int ans = 0;
        for (; x; x -= x & -x) {
            ans += c[x];
        }
        return ans;
    }
}

void dfs(int u, int fu) {
    g[u].insert(u);
    scan[u].push_back({u, id[u]});

    for (auto [v, _] : G.table[u]) {
        if (v == fu) continue;

        dfs(v, u);

        if (g[u].size() < g[v].size()) {
            std::swap(g[u], g[v]);
        }

        for (int x : g[v]) {
            auto it = g[u].lower_bound(x);
            if (it != g[u].end()) {
                scan[*it].push_back({x, id[u]});
            }
            if (it != g[u].begin()) {
                it --;
                scan[x].push_back({*it, id[u]});
            }
        }
        for (int x : g[v]) {
            g[u].insert(x);
        }
        // g[v].clear();
    }
}

int main() {
    std::ios::sync_with_stdio(0);
    std::cin.tie(0);

    std::cin >> n >> Q;

    G.init(n);
    for (int i = 1; i < n; i ++) {
        int x, y, z;
        std::cin >> x >> y >> z;

        G.add_edge(x, y, z), G.add_edge(y, x, z);
    }

    dfs_init(1, 0);

    for (int i = 1; i <= n; i ++) mapval[++ len] = dist[i];
    std::sort(mapval + 1, mapval + 1 + len);
    len = std::unique(mapval + 1, mapval + 1 + len) - mapval - 1;
    for (int i = 1; i <= n; i ++) id[i] = turn(dist[i]);

    dfs(1, 0);

    for (int i = 1; i <= Q; i ++) {
        int l, r;
        std::cin >> l >> r;

        qry[r].push_back({l, i});
    }

    for (int i = 1; i <= n; i ++) {
        for (auto [l, v] : scan[i]) {
            if (l > lst[v]) {
                if (lst[v]) BIT::add(lst[v], -1);
                BIT::add(l, +1);
                lst[v] = l;
            }
        }
        for (auto [l, id] : qry[i]) {
            ans[id] = BIT::ask(i) - BIT::ask(l - 1);
        }
    }

    for (int i = 1; i <= Q; i ++) {
        std::cout << ans[i] << '\n';
    }

    return 0;
}

/**
 *  心中无女人
 *  比赛自然神
 *  模板第一页
 *  忘掉心上人
**/

/*
10 1
9 1 -4
9 8 2
8 10 5
9 7 -3
1 4 2
10 2 5
10 5 -1
7 3 -3
10 6 5
1 2
*/
```