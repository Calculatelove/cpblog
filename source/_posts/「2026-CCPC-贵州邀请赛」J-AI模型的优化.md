---
title: 「2026 CCPC 贵州邀请赛」J. AI模型的优化
date: 2026-06-22 19:15:33
updated: 2026-06-22 19:15:33
categories: 邀请赛或省赛
tags:
  - DSU on tree
  - 重链剖分
---

# Description

Link：https://codeforces.com/gym/695551/problem/J

{% note default %}

给出一棵包含 $n$ 个点的树。

定义 $f(l, r)$ 表示包含 $l, \dots, r$ 的最小连通块的大小，求 $\sum_{1 \leq l \leq r \leq n} f(l, r)$。

数据范围：$1 \leq n \leq 10^5$。

时空限制：$1$s / $512$MiB。

{% endnote %}

<!-- more -->

# Solution

一开始往“支配对”上面想，属于是想歪了。因为很难处理出子树内对子树外的支配对。

注意到树上连通块的点数，等于边数 $+1$。所以我们可以考虑计算边的贡献。

## 算法一

相当于是要求，有多少个区间既有子树内的点又有子树外的点。

正难则反，可以用总区间数，减去“所有点都在子树内的区间个数”以及“所有点都在子树外的区间个数”。

以“所有点都在子树内的区间个数”为例，如果一个点在子树内就将其标记成 $1$，否则标记成 $0$。则一个长度为 $L$ 的全 $1$ 连续段，会产生 $\frac{L(L + 1)}{2}$ 的贡献。

于是可以使用 DSU on tree 维护将子树内所有点全都标记成 $1$ 的过程，使用 `std::set` 维护连续段。

时间复杂度 $\mathcal{O}(n \log^2 n)$。

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

const int N = 200100;

int n;
std::vector<std::vector<int>> G;

i64 calc(int x) {
    return 1ll * x * (x + 1) / 2;
}

struct segment {
    i64 sum;
    std::set<std::pair<int, int>> s;

    void all0() {
        sum = 0;
        s = {};
    }
    void all1() {
        sum = calc(n);
        s = {{1, n}};
    }

    void add(int x) {
        auto it = s.lower_bound({x, 0});
        int l = x, r = x;
        if (it != s.begin()) {
            auto itl = std::prev(it);
            auto [l1, r1] = *itl;
            if (r1 + 1 == x) {
                sum -= calc(r1 - l1 + 1);
                s.erase(itl);
                l = l1;
            }
        }
        if (it != s.end()) {
            auto [l2, r2] = *it;
            if (l2 - 1 == x) {
                sum -= calc(r2 - l2 + 1);
                s.erase(it);
                r = r2;
            }
        }
        sum += calc(r - l + 1);
        s.insert({l, r});
    }

    void dec(int x) {
        auto it = -- s.lower_bound({x + 1, 0});
        auto [l, r] = *it;

        sum -= calc(r - l + 1);
        s.erase(it);
        if (x > l) {
            sum += calc(x - l);
            s.insert({l, x - 1});
        }
        if (x < r) {
            sum += calc(r - x);
            s.insert({x + 1, r});
        }
    }
} si, so;

int sze[N], son[N];

void dfs_init(int u, int fu) {
    sze[u] = 1, son[u] = 0;
    for (int v : G[u]) {
        if (v == fu) {
            continue;
        }
        dfs_init(v, u);
        sze[u] += sze[v];
        if (sze[v] > sze[son[u]]) {
            son[u] = v;
        }
    }
}

i64 ans;

void add(int x) {
    si.add(x), so.dec(x);
}
void addTree(int u, int fu) {
    add(u);
    for (int v : G[u]) {
        if (v == fu) {
            continue;
        }
        addTree(v, u);
    }
}

void solve(int u, int fu, bool save) {
    for (int v : G[u]) {
        if (v == fu || v == son[u]) {
            continue;
        }
        solve(v, u, 0);
    }
    if (son[u]) {
        solve(son[u], u, 1);
    }

    for (int v : G[u]) {
        if (v == fu || v == son[u]) {
            continue;
        }
        addTree(v, u);
    }
    add(u);

    if (u > 1) {
        ans += 1ll * n * (n + 1) / 2 - si.sum - so.sum;
    }

    if (!save) {
        si.all0(), so.all1();
    }
}

int main() {
    std::ios::sync_with_stdio(0);
    std::cin.tie(0);

    std::cin >> n;

    G.assign(n + 1, {});
    for (int i = 1; i < n; i ++) {
        int x, y;
        std::cin >> x >> y;
        G[x].push_back(y);
        G[y].push_back(x);
    }

    dfs_init(1, 0);

    ans = 1ll * n * (n + 1) / 2;
    si.all0(), so.all1();
    solve(1, 0, 1);

    std::cout << ans << '\n';

    return 0;
}
```

## 算法二

还是计算边的贡献。对于某条边，如果点对 $(i, i + 1)$ 的其中一个点在子树内，另外一个点在子树外，那么称点对 $(i, i + 1)$ 是这条边的断点。

那么**一个区间既有子树内的点又有子树外的点，当且仅当这个区间内存在断点**。

注意到点对 $(i, i + 1)$，会对从 $i$ 到 $i + 1$ 的路径上的所有边 $e$ 都产生一个断点。设边 $e$ 的上一个断点位置为 $lst_e$，则新产生的贡献为 $(i - lst_e)(n - i)$。

相当于要支持路径 $lst_e$ 求和与路径 $lst_e$ 覆盖，使用重链剖分 + 线段树维护即可。

时间复杂度 $\mathcal{O}(n \log^2 n)$。