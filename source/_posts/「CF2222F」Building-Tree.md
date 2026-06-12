---
title: 「CF2222F」Building Tree
date: 2026-05-18 15:14:57
updated: 2026-05-18 15:14:57
categories: Codeforces
tags:
  - 数据结构
  - 分治
  - 缺一分治
---

# Description

Link：[CF2222F](https://codeforces.com/contest/2222/problem/F)

{% note default %}

给出一个包含 $n$ 个点 $m$ 条边的无向图，边带权。

对于任意路径，设其边权集合为 $S$，则该路径的长度为 $\mathrm{mex}(S)$。记 $\mathrm{dis}(u, v)$ 表示所有从 $u$ 到 $v$ 的路径中的最小路径长度。

现在有 $q$ 个关键点 $c_1, \dots, c_q$，连接 $c_u, c_v$ 的代价是 $\mathrm{dis}(c_u, c_v)$，求让这 $q$ 个关键点连通的最小代价总和。

数据范围：$1 \leq n, m \leq 3 \times 10^5$，$1 \leq q \leq n$，$0 \leq w \leq m$。

时空限制：$2.5$s / $512$MiB。

{% endnote %}

<!-- more -->

# Solution

考虑放宽限制，如果把权值为 $x$ 的所有边都删掉之后 $u, v$ 依然连通，那么可以认为新图中 $u, v$ 之间有一条权值为 $x$ 的边。由于求的是最小生成树，即使 $x$ 比真正的 $\mathrm{dis}(u, v)$ 更大也没有关系。

发现这个形式与缺一分治比较像。

考虑分治，设 $\mathrm{solve}(l, r)$ 表示当前加入了**边权不属于 $[l, r]$ 的所有边**。
- 向左侧递归时，加入边权属于 $[\mathrm{mid} + 1, r]$ 的所有边。
- 向右侧递归时，加入边权属于 $[l, \mathrm{mid}]$ 的所有边。
- 回溯时，将递归时加入的边撤销掉。

这样，递归到叶子结点 $[l, l]$ 时，相当于就是将权值为 $l$ 的所有边全都删掉了。

但显然不能递归到叶子结点 $[l, l]$ 的时候，再枚举任意两个点 $u, v$ 是否连通，尝试在新图中加边。

所以我们得考虑在递归的过程中加边，注意到：向左侧递归，在加入边权属于 $[\mathrm{mid} + 1, r]$ 的所有边时，如果 $u, v$ 在此刻连通了，那么意味着把权值属于 $[l, \mathrm{mid}]$ 的所有边都删掉之后 $u, v$ 依然连通，**于是我们可以认为新图中 $u, v$ 之间有一条权值为 $l$ 的边（向右侧递归也一样，只是在新图中加入一条边权为 $\mathrm{mid} + 1$ 的边）**。

在原图上维护可撤销并查集（每个集合需要额外维护一个代表特殊点），在新图上维护普通并查集。如果递归的过程中，原图中 $u, v$ 在此刻连通了，设 $u, v$ 所在集合的代表特殊点分别为 $p, q$，那么尝试在新图中连接 $p, q$。

时间复杂度 $\mathcal{O}(m \log^2 m)$。

```c++
#include <bits/stdc++.h>

using i64 = long long;

#define debug(a) std::cout << #a << '=' << (a) << ' '

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

const int N = 300100;

int n, m, Q;

std::vector<std::pair<int, int>> e[N];

int flag[N];

i64 ans;
struct nDSU {
    std::vector<int> fa;

    void init(int n) {
        fa.resize(n + 1);
        std::iota(fa.begin(), fa.end(), 0);
    }

    int get(int x) {
        return fa[x] == x ? x : fa[x] = get(fa[x]);
    }

    bool merge(int x, int y, int z) {
        int p = get(x), q = get(y);
        if (p == q) {
            return 0;
        }
        ans += z;
        fa[p] = q;
        return 1;
    }
} d1;

// 可撤销并查集
struct DSU {
    std::vector<int> sze, key;
    std::vector<int> fa;
    std::vector<std::array<int, 3>> his;

    DSU() {}
    DSU(int n) {
        init(n);
    }

    void init(int n) {
        sze.assign(n + 1, 1);
        key.assign(n + 1, 0);
        fa.resize(n + 1);
        std::iota(fa.begin(), fa.end(), 0);
        his.clear();
    }

    int get(int x) {
        return fa[x] == x ? x : get(fa[x]); // 按秩合并不能路径压缩
    }

    bool merge(int x, int y, int z) {
        int p = get(x), q = get(y);
        if (p == q) {
            return 0;
        }
        if (sze[p] > sze[q]) {
            std::swap(p, q);
        }
        his.push_back({p, q, key[q]});
        if (key[p] && key[q]) {
            d1.merge(key[p], key[q], z);
        }
        sze[q] += sze[p], key[q] = key[p] ? key[p] : key[q], fa[p] = q;
        return 1;
    }

    int time() {
        return his.size();
    }
    
    void revert(int tim) {
        while (his.size() > tim) {
            auto [p, q, k] = his.back(); his.pop_back();
            sze[q] -= sze[p], key[q] = k, fa[p] = p;
        }
    }
} d0;

void solve(int l, int r) {
    if (l == r) {
        return;
    }
    int mid = (l + r) >> 1;

    int tim = d0.time();
    for (int i = mid + 1; i <= r; i ++) {
        for (auto [x, y] : e[i]) {
            d0.merge(x, y, l);
        }
    }
    solve(l, mid);

    d0.revert(tim);
    for (int i = l; i <= mid; i ++) {
        for (auto [x, y] : e[i]) {
            d0.merge(x, y, mid + 1);
        }
    }
    solve(mid + 1, r);
}

void work() {
    std::cin >> n >> m >> Q;

    for (int i = 0; i <= m; i ++) {
        e[i].clear();
    }
    for (int i = 1; i <= m; i ++) {
        int x, y, z;
        std::cin >> x >> y >> z;
        e[z].push_back({x, y});
    }

    d0.init(n), d1.init(n), ans = 0;
    for (int i = 1; i <= n; i ++) {
        flag[i] = 0;
    }
    for (int i = 1; i <= Q; i ++) {
        int x;
        std::cin >> x;
        flag[x] = 1;
        d0.key[x] = x;
    }

    solve(0, m);

    int p = 0;
    for (int i = 1; i <= n; i ++) {
        if (flag[i]) {
            int x = d1.get(i);
            if (!p) {
                p = x;
            } else if (x != p) {
                std::cout << -1 << '\n';
                return;
            }
        }
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

/*
1
10 12 5
8 2 0
10 1 1
6 10 0
4 2 3
4 3 2
3 8 3
10 7 0
4 5 1
6 4 0
2 10 0
1 3 0
7 9 0
6 1 7 9 5
*/
```