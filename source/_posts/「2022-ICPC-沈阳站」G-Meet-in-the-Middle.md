---
title: 「2022 ICPC 沈阳站」G. Meet in the Middle
date: 2026-03-09 17:45:49
updated: 2026-03-09 17:45:49
categories: ICPC
tags:
  - 线段树
  - 树的直径
---

# Description

Link：[gym104160 G](https://codeforces.com/gym/104160/problem/G)

{% note default %}

给出两棵大小均为 $n$ 的树，节点编号均为 $1 \sim n$，边带正权。

有 $Q$ 次询问，每次询问给出两个正整数 $a, b$ ($1 \leq a, b \leq n$)，你需要找到一个节点 $x$，使得 $x$ 在第一棵树上到 $a$ 的距离加上 $x$ 在第二棵树上到 $b$ 的距离之和最大，你只需要求出这个最大值即可。

数据范围：$1 \leq n \leq 10^5$，$1 \leq q \leq 5 \times 10^5$，$1 \leq w \leq 10^9$。

时空限制：$5$s / $512$MiB。

{% endnote %}

<!-- more -->

# Solution

考虑离线处理询问。在第一棵树上 dfs 询问点 $a$，假设在第二棵树上的每个点 $x$ 下面都挂一个新点 $x'$ 且边权为 $\mathrm{dist}_1(a, x)$，那么问题就转化为了查询**新树上关于 $b$ 的最远点**。最远点必定是新树直径的两个端点之一，问题就转化成了动态维护新树的直径。

对第一棵树的 dfs 序建线段树，线段树上的每个节点都维护了**第一棵树上的 dfs 序在该区间内的所有点，对应到第二棵树上的点集直径（只需要维护直径的两个端点，以及对应的 $\mathrm{dist}_1(a, x)$ 即可）**。两个点集直径的合并是经典套路。

当 $a$ 在第一棵树上移动时（例如走向某个子树 $v$），子树 $v$ 内的所有点的 $\mathrm{dist}_1(a, x)$ 都会全体减去边长 $w$，子树 $v$ 外的所有点都会全体加上边长 $w$。对应到线段树上就是区间加，**我们只需在线段树上走的同时触发区间点集直径的更新即可**，那些未触发更新的区间只是整体加减一个值，并不改变直径的端点，只需打上懒标记即可。

时间复杂度 $\mathcal{O}(n \log n + Q)$。

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

const int N = 100100, MaxQ = 500100;

int n, Q;
std::vector<std::vector< std::pair<int, int> >> G1, G2;

i64 dist1[N];

int dfsClock, dfn[N], idx[N];
int sze[N];

void dfs1_init(int u, int fu) {
    dfsClock ++;
    dfn[u] = dfsClock, idx[dfsClock] = u;

    sze[u] = 1;
    for (auto [v, w] : G1[u]) {
        if (v == fu) {
            continue;
        }
        dist1[v] = dist1[u] + w;
        dfs1_init(v, u);
        sze[u] += sze[v];
    }
}

int dep2[N];
i64 dist2[N];

int Fir[N];
std::vector<int> eul;

void dfs2_init(int u, int fu) {
    dep2[u] = dep2[fu] + 1;
    Fir[u] = eul.size(), eul.push_back(u);
    for (auto [v, w] : G2[u]) {
        if (v == fu) {
            continue;
        }
        dist2[v] = dist2[u] + w;
        dfs2_init(v, u);
        eul.push_back(u);
    }
}

namespace eulST {
    int n, t;
    std::vector<std::vector<int>> f;

    inline int op(int x, int y) {
        return dep2[x] < dep2[y] ? x : y;
    }

    void init() {
        n = eul.size(), t = std::__lg(n);
        f.assign(t + 1, std::vector<int>(n, 0));

        for (int i = 0; i < n; i ++) {
            f[0][i] = eul[i];
        }
        for (int j = 1; j <= t; j ++) {
            for (int i = 0; i + (1 << j) - 1 < n; i ++) {
                f[j][i] = op(f[j - 1][i], f[j - 1][i + (1 << (j - 1))]);
            }
        }
    }

    int ask(int l, int r) {
        int k = std::__lg(r - l + 1);
        return op(f[k][l], f[k][r - (1 << k) + 1]);
    }
}

int lca(int x, int y) {
    auto [l, r] = std::minmax(Fir[x], Fir[y]);
    return eulST::ask(l, r);
}
i64 distance2(int x, int y) {
    return dist2[x] + dist2[y] - 2 * dist2[lca(x, y)];
}

struct Info {
    int p, q;
    i64 a, b;
    
    Info() {}
    Info(int _p, int _q, i64 _a, i64 _b) : p(_p), q(_q), a(_a), b(_b) {}

    void mk_add(i64 x) {
        a += x, b += x;
    }

    friend Info operator + (Info lhs, Info rhs) {
        Info self;
        i64 mx = 0;

        auto upd = [&] (int p, int q, i64 a, i64 b) {
            i64 x = a + b + distance2(p, q);
            if (x > mx) {
                mx = x;
                self = Info(p, q, a, b);
            }
        };
        upd(lhs.p, lhs.q, lhs.a, lhs.b);
        upd(rhs.p, rhs.q, rhs.a, rhs.b);
        upd(lhs.p, rhs.p, lhs.a, rhs.a);
        upd(lhs.p, rhs.q, lhs.a, rhs.b);
        upd(lhs.q, rhs.p, lhs.b, rhs.a);
        upd(lhs.q, rhs.q, lhs.b, rhs.b);

        return self;
    }
};

namespace SGT {
    struct node {
        Info info;
        i64 add;
        void mk_add(i64 x) {
            info.mk_add(x);
            add += x;
        }
    } t[N * 4];

    void upd(int p) {
        t[p].info = t[p * 2].info + t[p * 2 + 1].info;
    }

    void spread(int p) {
        if (t[p].add) {
            t[p * 2].mk_add(t[p].add), t[p * 2 + 1].mk_add(t[p].add);
            t[p].add = 0;
        }
    }

    void build(int p, int l, int r) {
        if (l == r) {
            int u = idx[l];
            t[p].info = Info(u, u, dist1[u], dist1[u]);
            return;
        }
        int mid = (l + r) >> 1;
        build(p * 2, l, mid), build(p * 2 + 1, mid + 1, r);
        upd(p);
    }

    void change(int p, int l, int r, int s, int e, i64 x) {
        if (s <= l && r <= e) {
            t[p].mk_add(x);
            return;
        }
        spread(p);
        int mid = (l + r) >> 1;
        if (s <= mid) {
            change(p * 2, l, mid, s, e, x);
        }
        if (mid < e) {
            change(p * 2 + 1, mid + 1, r, s, e, x);
        }
        upd(p);
    }

    i64 ask(int x) {
        auto [p, q, a, b] = t[1].info;
        i64 mx = 0;
        chmax(mx, a + distance2(x, p));
        chmax(mx, b + distance2(x, q));
        return mx;
    }
}

std::vector<std::pair<int, int>> lay[N];
i64 ans[MaxQ];

void solve(int u, int fu) {
    for (auto [p, id] : lay[u]) {
        ans[id] = SGT::ask(p);
    }

    for (auto [v, w] : G1[u]) {
        if (v == fu) {
            continue;
        }
        int l = dfn[v], r = dfn[v] + sze[v] - 1;

        SGT::change(1, 1, n, l, r, -w);
        if (l > 1) SGT::change(1, 1, n, 1, l - 1, +w);
        if (r < n) SGT::change(1, 1, n, r + 1, n, +w);

        solve(v, u);

        SGT::change(1, 1, n, l, r, +w);
        if (l > 1) SGT::change(1, 1, n, 1, l - 1, -w);
        if (r < n) SGT::change(1, 1, n, r + 1, n, -w);
    }
}

int main() {
    std::ios::sync_with_stdio(0);
    std::cin.tie(0);

    std::cin >> n >> Q;

    G1.assign(n + 1, {}), G2.assign(n + 1, {});
    for (int i = 1; i < n; i ++) {
        int x, y, z;
        std::cin >> x >> y >> z;
        G1[x].push_back({y, z}), G1[y].push_back({x, z});
    }
    for (int i = 1; i < n; i ++) {
        int x, y, z;
        std::cin >> x >> y >> z;
        G2[x].push_back({y, z}), G2[y].push_back({x, z});
    }

    dfs1_init(1, 0);
    dfs2_init(1, 0);
    eulST::init();

    SGT::build(1, 1, n);

    for (int i = 1; i <= Q; i ++) {
        int a, b;
        std::cin >> a >> b;
        lay[a].push_back({b, i});
    }

    solve(1, 0);

    for (int i = 1; i <= Q; i ++) {
        std::cout << ans[i] << '\n';
    }

    return 0;
}
```