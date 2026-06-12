---
title: 「2022 上海市赛」F. Forest of Magic
date: 2025-03-13 00:00:02
updated: 2026-03-04 00:00:02
categories: 邀请赛或省赛
tags:
  - 「数据结构」
  - 分块
  - 主席树
---

# Description

Link：[gym103931 F](https://codeforces.com/gym/103931/problem/F)

{% note default %}

给出一棵包含 $n$ 个点的树，编号 $1 \sim n$。根节点为 $1$。

有 $Q$ 次操作，每次操作形如以下的三种之一：
- `1 u`：设本次操作之前共有 $n'$ 个点，则新加入一个编号为 $n' + 1$ 的点，并且新点有一条连向 $u$ 的无向边。
- `2 u v c k`：对于在从 $u$ 到 $v$ 的简单路径上的所有点，都会增加 $k$ 个类型 $c$ 的物品。
- `3 u c`：查询点 $u$ 的子树内，有多少个物品的类型 $\leq c$。

**本题强制在线。**

数据范围：$1 \leq n \leq 3 \times 10^4$，$0 \leq Q \leq 10^5$，节点总数不超过 $5 \times 10^4$，$1 \leq k \leq 10^7$，$1 \leq c \leq 10^9$。

时空限制：$8$s / $1024$MiB。

{% endnote %}

<!-- more -->

# Solution

对于一次修改操作 $(x, y, c, k)$，设 $z = \mathrm{LCA}(x, y)$，我们将其拆成四条从根节点开始的路径 $(1, x, c, k), (1, y, c, k), (1, z, c, -k), (1, \mathrm{fa}_z, c, -k)$。

那么一次修改 $(1, u, c_1, k)$ 对询问 $(v, c_2)$ 的贡献为 $[u \in \mathrm{subtree}(v)][c_1 \leq c_2] \cdot k(\mathrm{dep}_{u} - \mathrm{dep}_{v} + 1)$。

这是一个经典的二维数点问题，如果问题是静态的（没有操作 1 且操作 2 提前给出），我们可以在 dfs 序上使用主席树维护。

那么对于动态的原问题，考虑**定期重构**，设每 $B$ 个操作重构一次。在上一次重构前的修改在 dfs 序上使用主席树计算，剩余的修改暴力计算。

这里要注意的点是：如何实时判断 $x$ 是否是 $y$ 的祖先？我们再维护一个二维数组 `inc[][]` 表示上一次重构之后的新点的祖先关系，这个二维数组的大小显然是 $\mathcal{O}(B^2)$ 的。

不妨设 $n, Q$ 同阶。则时间复杂度为 $\mathcal{O}\left(\frac{n^2 \log n}{B} + nB + n \log n\right)$，取 $B = \sqrt{n \log n}$，时间复杂度 $\mathcal{O}(n\sqrt{n \log n})$。

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

const int N = 50010, MaxM = 100100;
const int sup = 1e9;

const int B = 2000;
const int Blen = B + 10;

int n, m;
int lstn;

std::vector<std::vector<int>> G;

struct Info {
    i64 c, s;
    Info() {}
    Info(i64 _c, i64 _s) : c(_c), s(_s) {}
    friend Info operator + (const Info &lhs, const Info &rhs) {
        return Info(lhs.c + rhs.c, lhs.s + rhs.s);
    }
    friend Info operator - (const Info &lhs, const Info &rhs) {
        return Info(lhs.c - rhs.c, lhs.s - rhs.s);
    }
    Info &operator += (const Info &rhs) {
        return (*this) = (*this) + rhs;
    }
};

int root[N];
namespace SGT {
    const int pond = 10001000;

    int nodeCount;
    struct node {
        int lc, rc;
        Info info;
    } t[pond];

    void init() {
        nodeCount = 0;
    }

    void insert(int &p, int q, int l, int r, int x, Info y) {
        p = ++ nodeCount, t[p] = t[q], t[p].info += y;
        if (l == r) return;
        int mid = (l + r) >> 1;
        if (x <= mid) {
            insert(t[p].lc, t[q].lc, l, mid, x, y);
        } else {
            insert(t[p].rc, t[q].rc, mid + 1, r, x, y);
        }
    }

    Info ask(int p, int l, int r, int x) {
        if (l == r) return t[p].info;
        int mid = (l + r) >> 1;
        if (x <= mid) {
            return ask(t[p].lc, l, mid, x);
        } else {
            return t[t[p].lc].info + ask(t[p].rc, mid + 1, r, x);
        }
    }
}

int dep[N];
int anc[18][N];

void dfs_start(int u, int fu) {
    dep[u] = dep[fu] + 1;
    anc[0][u] = fu;
    for (int i = 1; i <= 17; i ++) {
        anc[i][u] = anc[i - 1][anc[i - 1][u]];
    }

    for (int v : G[u]) {
        if (v == fu) {
            continue;
        }
        dfs_start(v, u);
    }
}

int lca(int x, int y) {
    if (dep[x] > dep[y]) std::swap(x, y);
    for (int i = 17; i >= 0; i --)
        if (dep[x] <= dep[y] - (1 << i)) y = anc[i][y];
    if (x == y) return x;
    for (int i = 17; i >= 0; i --)
        if (anc[i][x] ^ anc[i][y]) x = anc[i][x], y = anc[i][y];
    return anc[0][x];
}

int dfsClock, dfn[N], idx[N];
int sze[N];

void dfs_init(int u, int fu) {
    dfsClock ++;
    dfn[u] = dfsClock, idx[dfsClock] = u;

    sze[u] = 1;
    for (int v : G[u]) {
        if (v == fu) {
            continue;
        }
        dfs_init(v, u);
        sze[u] += sze[v];
    }
}

int fa[N];
int inc[Blen][Blen];

int contain(int x, int y) {
    if (x <= lstn) {
        y = fa[y];
        return dfn[x] <= dfn[y] && dfn[y] <= dfn[x] + sze[x] - 1;
    } else {
        if (y <= lstn) {
            return 0;
        } else {
            return inc[y - lstn][x - lstn];
        }
    }
}

std::vector<std::array<int, 3>> lstop[N];
// lstop[u] : {c, k, opt}

std::vector<std::array<int, 5>> curop;
// curop : {x, y, z, c, k}

void rebuild(int del = 1) {
    if (del) {
        for (int i = lstn + 1; i <= n; i ++) {
            int x = i;
            while (x > lstn) {
                inc[i - lstn][x - lstn] = 0;
                x = anc[0][x];
            }
        }
    }

    for (auto [x, y, z, c, k] : curop) {
        lstop[x].push_back({c, k, +1});
        lstop[y].push_back({c, k, +1});
        lstop[z].push_back({c, k, -1});
        if (anc[0][z]) lstop[anc[0][z]].push_back({c, k, -1});
    }
    std::vector<std::array<int, 5>>().swap(curop);

    lstn = n;
    for (int i = 1; i <= n; i ++) {
        fa[i] = i;
    }

    dfsClock = 0;
    dfs_init(1, 0);

    SGT::init();
    for (int i = 1; i <= n; i ++) {
        root[i] = root[i - 1];

        int u = idx[i];
        for (auto [c, k, opt] : lstop[u]) {
            Info info(opt * k, 1ll * opt * k * dep[u]);
            SGT::insert(root[i], root[i], 1, sup, c, info);
        }
    }
}

i64 ask(int u, int c) {
    i64 ans = 0;
    if (u <= lstn) {
        int l = dfn[u], r = dfn[u] + sze[u] - 1;
        Info info = SGT::ask(root[r], 1, sup, c) - SGT::ask(root[l - 1], 1, sup, c);
        ans += info.s - info.c * (dep[u] - 1);
        // debug(info.s), debug(info.c) << '\n';
    }

    for (auto [x, y, z, qc, k] : curop) {
        if (qc > c) {
            continue;
        }
        if (contain(u, z)) {
            ans += 1ll * (dep[x] + dep[y] - dep[z] * 2 + 1) * k;
        } else {
            if (contain(u, x)) {
                ans += 1ll * (dep[x] - dep[u] + 1) * k;
            }
            if (contain(u, y)) {
                ans += 1ll * (dep[y] - dep[u] + 1) * k;
            }
        }
    }
    return ans;
}

const i64 mod = 1LL << 31;

int main() {
    std::ios::sync_with_stdio(0);
    std::cin.tie(0);

    std::cin >> n >> m;    

    G.assign(n + m + 1, {});
    for (int i = 1; i < n; i ++) {
        int x, y;
        std::cin >> x >> y;

        G[x].push_back(y), G[y].push_back(x);
    }

    dfs_start(1, 0);

    rebuild(0);

    int lastans = 0;
    for (int qid = 1; qid <= m; qid ++) {
        int opt, u, v, c, k;
        std::cin >> opt >> u;
        u ^= lastans;

        if (opt == 1) {
            int p = ++ n;
            G[u].push_back(p);

            fa[p] = fa[u];
            dep[p] = dep[u] + 1;
            anc[0][p] = u;
            for (int i = 1; i <= 17; i ++) {
                anc[i][p] = anc[i - 1][anc[i - 1][p]];
            }

            int x = p;
            while (x > lstn) {
                inc[p - lstn][x - lstn] = 1;
                x = anc[0][x];
            }
        } else if (opt == 2) {
            std::cin >> v >> c >> k;
            v ^= lastans, c ^= lastans, k ^= lastans;

            curop.push_back({u, v, lca(u, v), c, k});
        } else {
            std::cin >> c;
            c ^= lastans;

            i64 ans = ask(u, c);
            std::cout << ans << '\n';

            lastans = ans % mod;
        }

        if (qid % B == 0) {
            rebuild();
        }
    }

    return 0;
}

/*
3 2
1 2
1 3
2 2 3 1 4
3 1 1
*/
```