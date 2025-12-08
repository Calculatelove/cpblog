---
title: 「CF2174D」Secret Message
date: 2025-12-08 16:51:14
updated: 2025-12-08 16:51:14
categories: Codeforces
tags:
  - 树上问题
---

# Description

Link：[CF2174D](https://codeforces.com/contest/2174/problem/D)

{% note default %}

给出一个包含 $n$ 个点 $m$ 条边的带权无向图。

你需要找出 $n - 1$ 条边，使得这些边的权值之和最小，且不构成一棵树。

数据范围：$2 \leq n \leq 2 \times 10^5$，$n - 1 \leq m \leq 2 \times 10^5$，$1 \leq w_i \leq 10^9$。

时空限制：$2$s / $256$MiB。

{% endnote %}

<!-- more -->

# Solution

首先将所有边按照边权从小到大排序为 $e_1, e_2, \cdots, e_m$，记权值分别为 $w_1, w_2, \cdots, w_m$。

首先考虑前 $n - 1$ 条边，如果不构成一棵树则结束。否则我们可以得到前 $n - 1$ 条边构成的生成树，考虑在该生成树上替换若干条边。

#### 替换一条边

对于一条非树边 $(x, y, z)$，我们只能替换**不在从 $x$ 到 $y$ 路径上的边**。所以现在要求的是，不在从 $x$ 到 $y$ 路径上的最大边。

一个简单的做法是，取出最大的树边 $(x_m, y_m, z_m)$，若最大树边不在从 $x$ 到 $y$ 的路径上，则我们要求的就是这条边。

否则将该最大树边断开，得到两个根节点分别为 $x_m, y_m$ 的树。相当于是**给定根节点，查询不在从根到 $x$ 的路径上的最大边**。可以使用类似换根 dp 的方法统计出。

（邪修：使用树剖维护。每次查询时，将 $x$ 到 $y$ 路径上的所有边加上 $-\infty$，然后查询全局最大值）

#### 替换两条以上的边

对于替换两条以上的边，我们可以证明 $\mathrm{sum} - w_{n - 1} - w_{n - 2} + w_n + w_{n + 1}$ 是答案的一个上界。其实际意义也就是将 $e_{n - 1}, e_{n - 2}$ 替换成 $e_n, e_{n + 1}$。

首先，如果有 $e_{n - 1}$ 或 $e_{n - 2}$ 不在 $e_n$ 或 $e_{n + 1}$ 的管辖路径上，则必定有一个更优秀的 “替换一条边” 的方案。

所以我们假设 $e_{n - 1}, e_{n - 2}$ 都在 $e_n$ 与 $e_{n + 1}$ 的管辖路径上，此时断开 $e_{n - 1}, e_{n - 2}$ 会形成三个连通块，然而 $e_n, e_{n + 1}$ 连接的是同一个连通块（可以画图理解），最后会剩下两个连通块，所以此时的方案是合法的。显然该方案是 “替换两条以上的边” 之中代价最小的。

时间复杂度 $\mathcal{O}(n + m \log m)$。

```c++
#include <bits/stdc++.h>

using i64 = long long;
using u64 = unsigned long long;

#define debug(a) std::cout << #a << "=" << (a) << ' '

template <class T>
inline void chmin(T &x, const T &y) {
    if (x > y) {
        x = y;
    }
}
template <class T>
inline void chmax(T &x, const T &y) {
    if (x < y) {
        x = y;
    }
}

const int N = 200100, M = 200100;
const i64 inf = 1e18;

int n, m;
std::tuple<int, int, int> e[M];

int fa[N];
int get(int x) {
    return fa[x] == x ? x : fa[x] = get(fa[x]);
}

struct Graph {
    std::vector< std::vector< std::pair<int, int> > > table;

    void init(int _n) {
        table.assign(_n + 1, {});
    }

    void add_edge(int u, int v, int w) {
        table[u].push_back({v, w});
    }
} G;

i64 f[N], g[N];

void dp1(int u, int fu) {
    for (auto [v, w] : G.table[u]) {
        if (v == fu) {
            continue;
        }
        f[v] = w, dp1(v, u);
        chmax(f[u], f[v]);
    }
}

void dp2(int u, int fu, i64 pre) {
    i64 fir = -inf, sec = -inf;
    g[u] = pre;
    for (auto [v, w] : G.table[u]) {
        if (v == fu) {
            continue;
        }
        chmax(g[u], f[v]);
        if (f[v] > fir) {
            sec = fir, fir = f[v];
        } else {
            chmax(sec, f[v]);
        }
    }
    for (auto [v, w] : G.table[u]) {
        if (v == fu) {
            continue;
        }
        dp2(v, u, std::max(pre, f[v] == fir ? sec : fir));
    }
}

void work() {
    std::cin >> n >> m;

    for (int i = 1; i <= m; i ++) {
        int x, y, z;
        std::cin >> x >> y >> z;

        e[i] = {z, x, y};
    }

    std::sort(e + 1, e + 1 + m);

    i64 ans = inf;
    i64 sum = 0;
    for (int i = 1; i < n; i ++) {
        sum += std::get<0>(e[i]);
    }

    std::iota(fa + 1, fa + 1 + n, 1);
    for (int i = 1; i < n; i ++) {
        auto [z, x, y] = e[i];

        int p = get(x), q = get(y);
        if (p == q) {
            std::cout << sum << '\n';
            return;
        }

        fa[p] = q;
    }

    G.init(n);

    std::iota(fa + 1, fa + 1 + n, 1);
    for (int i = 1; i < n - 1; i ++) {
        auto [z, x, y] = e[i];

        fa[get(x)] = get(y);
        G.add_edge(x, y, z), G.add_edge(y, x, z);
    }

    for (int i = 1; i <= n; i ++) {
        f[i] = g[i] = -inf;
    }

    auto [mz, rx, ry] = e[n - 1];
    dp1(rx, 0), dp2(rx, 0, -inf);
    dp1(ry, 0), dp2(ry, 0, -inf);

    for (int i = n; i <= m; i ++) {
        auto [z, x, y] = e[i];

        if (get(x) == get(y)) {
            chmin(ans, sum - mz + z);
        } else {
            chmin(ans, sum - std::max(g[x], g[y]) + z);
        }
    }

    if (n >= 3 && m >= n + 1) {
        chmin(ans, sum - std::get<0>(e[n - 1]) - std::get<0>(e[n - 2]) + std::get<0>(e[n]) + std::get<0>(e[n + 1]));
    }

    if (ans == inf) {
        ans = -1;
    }
    std::cout << ans << '\n';
}

int main() {
    std::ios::sync_with_stdio(0);
    std::cin.tie(0);

    int T;
    std::cin >> T;
    
    while (T --) {
        work();
    }

    return 0;
}

/**
 *  心中无女人
 *  比赛自然神
 *  模板第一页
 *  忘掉心上人
**/
```