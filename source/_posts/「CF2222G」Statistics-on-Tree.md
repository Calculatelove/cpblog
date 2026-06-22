---
title: 「CF2222G」Statistics on Tree
date: 2026-05-18 16:14:57
updated: 2026-05-18 16:14:57
categories: Codeforces
tags:
  - 树上问题
---

# Description

Link：[CF2222G](https://codeforces.com/contest/2222/problem/G)

{% note default %}

给出一个大小为 $n$ 的树，定义点对 $(u, v)$ ($1 \leq u \leq v \leq n$) 的价值为：删除从 $u$ 到 $v$ 的路径上的**所有边**之后，最大连通块的大小。

对于每个 $i$ ($1 \leq i \leq n$)，求出价值等于 $i$ 的点对 $(u, v)$ 的数量。

数据范围：$1 \leq n \leq 10^5$。

时空限制：$2$s / $512$MiB。

{% endnote %}

<!-- more -->

# Solution

在定根的情况下，枚举点对 $(u, v)$ 的 $\mathrm{lca}(u, v) = z$，定义：
- $a_u$：删除 $z$ 到 $u$ 的所有边之后，不包含 $z$ 的最大连通块大小。
- $b_u$：$z$ 到 $u$ 路径上第二个点的子树大小。

那么点对 $(u, v)$ 的价值为 $\max(a_u, a_v, n - b_u - b_v)$。发现直接做的话非常难统计，因为这都已经比卷积更加严格了。

**考虑以树的重心为根**，树的重心有一个很好的性质：**所有子树的大小都不超过 $\frac{n}{2}$**。

这样的话，当点对的 $\mathrm{lca}(u, v)$ 不等于根时，外子树（包含 $z$ 的连通块）的大小必定超过 $\frac{n}{2}$，所以此时一定是外子树更大。所以当 $\mathrm{lca}(u, v)$ 不等于根时，点对 $(u, v)$ 的价值为 $n - b_u - b_v$。

这是一个卷积的形式。对于 $z$ 的每一个儿子 $x$，相当于是给 $sz_x$ 次项加上 $sz_x$，然后进行卷积。NTT 显然过不去，但是这个多项式比较稀疏，可以直接进行暴力多项式乘法（前提是要先去重）！可以证明时间复杂度是 $\mathcal{O}(n \log n)$ 的：
- 对整棵树进行重链剖分，显然**轻儿子的子树大小之和不超过 $\mathcal{O}(n \log n)$**。
- 轻儿子与轻儿子之间的复杂度：设 $z$ 的所有轻儿子的子树大小之和为 $k$，则本质不同的子树大小不超过 $\mathcal{O}(\sqrt{k})$，于是暴力多项式乘法的复杂度为 $\mathcal{O}(k)$。于是总和不超过 $\mathcal{O}(n \log n)$。
- 重儿子与轻儿子之间的复杂度：不超过点 $z$ 的度数。于是总和不超过 $\mathcal{O}(n)$。

还要额外统计一下 $\mathrm{lca}(u, v)$ 等于根的贡献。

先考虑 $\max(a_u, a_v)$ 决定了点对价值的情况，根据定义显然有 $a_u \leq b_u$，假设 $a_u \geq n - b_u - b_v$（不妨设 $a_u \geq a_v$）那么有 $2b_u + b_v \geq n$，所以必有 $b_u \geq \frac{n}{3}$ 或 $b_v \geq \frac{n}{3}$。

也就是说，当 $\max(a_u, a_v)$ 决定了点对 $(u, v)$ 的价值时，$u, v$ 的其中一个必定位于大小超过 $\frac{n}{3}$ 的子树内，然而这样的子树不超过两个。可以枚举这样的大子树去统计贡献，相当于是做一次二维数点。

再考虑 $n - b_u - b_v$ 决定了点对价值的情况，对于每一种 $b_u$，用一个 `std::vector` 存放 $b_u$ 相同的所有的 $a$ 值。依然去重之后暴力去枚举 $b_u, b_v$ 的值，那么此时要求 $a_u, a_v < n - b_u - b_v$，可以在各自的 `std::vector` 里面二分数出有多少个符合条件的 $a$。

注意来自同一子树的贡献还要额外排除。

细节较多。

时间复杂度 $\mathcal{O}(n \log n)$。

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

const int N = 100100;

int n;
std::vector<std::vector<int>> G;

int sz[N], mp[N];
int rt;

void getSize(int u, int fu) {
    sz[u] = 1;
    for (int v : G[u]) {
        if (v == fu) {
            continue;
        }
        getSize(v, u);
        sz[u] += sz[v];
    }
}
void getRoot(int u, int fu) {
    mp[u] = 0;
    for (int v : G[u]) {
        if (v == fu) {
            continue;
        }
        getRoot(v, u);
        chmax(mp[u], sz[v]);
    }
    chmax(mp[u], n - sz[u]);
    if (!rt || mp[u] < mp[rt]) {
        rt = u;
    }
}

i64 ans[N];

void solve(int u, int fu) {
    std::vector<std::pair<int, int>> tmp, seq;
    for (int v : G[u]) {
        if (v == fu) {
            continue;
        }
        solve(v, u);

        if (u == rt) {
            continue;
        }

        ans[n - sz[v]] += sz[v]; // v 在 u 的子树内
        ans[n - 2 * sz[v]] -= 1ll * sz[v] * (sz[v] - 1) / 2; // 容斥
        tmp.push_back({sz[v], sz[v]});
    }

    if (u == rt) {
        return;
    }

    std::sort(tmp.begin(), tmp.end());
    for (auto [v, c] : tmp) {
        if (!seq.size() || seq.back().first != v) {
            seq.push_back({v, c});
        } else {
            seq.back().second += c;
        }
    }

    for (int i = 0; i < seq.size(); i ++) {
        for (int j = i; j < seq.size(); j ++) {
            if (i == j) {
                ans[n - 2 * seq[i].first] += 1ll * seq[i].second * (seq[i].second - 1) / 2;
            } else {
                ans[n - seq[i].first - seq[j].first] += 1ll * seq[i].second * seq[j].second;
            }
        }
    }
}

int a[N], b[N];
void dfs_init(int u, int fu, int mp, int bel) {
    a[u] = std::max(mp, sz[u]), b[u] = bel;
    for (int v : G[u]) {
        if (v == fu) {
            continue;
        }
        dfs_init(v, u, std::max(mp, sz[u] - sz[v]), bel);
    }
}

std::vector<int> sa[N], tmp;
void find(int u, int fu) {
    tmp.push_back(a[u]);
    for (int v : G[u]) {
        if (v == fu) {
            continue;
        }
        find(v, u);
    }
}

struct BIT {
    int c[N];

    void init() {
        for (int i = 1; i <= n; i ++) {
            c[i] = 0;
        }
    }

    void add(int x, int y) {
        for (; x; x -= x & -x) {
            c[x] += y;
        }
    }

    int ask(int x) {
        int ans = 0;
        for (; x <= n; x += x & -x) {
            ans += c[x];
        }
        return ans;
    }
} bit[2];

std::vector<std::pair<int, int>> id;
void get(int u, int fu, int type) {
    id.push_back({u, type});
    for (int v : G[u]) {
        if (v == fu) {
            continue;
        }
        get(v, u, type);
    }
}

void work() {
    std::cin >> n;
    
    G.assign(n + 1, {});
    for (int i = 1; i < n; i ++) {
        int x, y;
        std::cin >> x >> y;
        G[x].push_back(y);
        G[y].push_back(x);
    }

    getSize(1, 0);
    rt = 0;
    getRoot(1, 0);
    getSize(rt, 0);

    for (int i = 1; i <= n; i ++) {
        ans[i] = 0;
    }

    ans[n] += n; // u = v;

    solve(rt, 0);
    // debug(rt) << '\n';

    for (int v : G[rt]) {
        dfs_init(v, rt, 0, sz[v]);
    }
    for (int i = 1; i <= n; i ++) {
        if (i != rt) {
            ans[std::max(a[i], n - b[i])] ++; // u = rt, v 在 rt 的子树内
            // debug(i), debug(a[i]), debug(b[i]) << '\n';
        }
    }

    for (int i = 1; i <= n; i ++) {
        sa[i].clear();
    }
    for (int i = 1; i <= n; i ++) {
        if (i != rt) {
            sa[b[i]].push_back(a[i]);
        }
    }
    for (int i = 1; i <= n; i ++) {
        std::sort(sa[i].begin(), sa[i].end());
    }

    std::vector<int> seq;
    for (int i = 1; i <= n; i ++) {
        if (sa[i].size()) {
            seq.push_back(i);
        }
    }
    for (int i = 0; i < seq.size(); i ++) {
        for (int j = i; j < seq.size(); j ++) {
            int v1 = seq[i], v2 = seq[j];
            int x = n - v1 - v2;

            int c1 = std::lower_bound(sa[v1].begin(), sa[v1].end(), x) - sa[v1].begin();
            int c2 = std::lower_bound(sa[v2].begin(), sa[v2].end(), x) - sa[v2].begin();
            if (v1 != v2) {
                ans[x] += 1ll * c1 * c2;
            } else {
                ans[x] += 1ll * c1 * (c1 - 1) / 2;
            }
        }
    }

    for (int v : G[rt]) {
        tmp.clear();
        find(v, rt);

        std::sort(tmp.begin(), tmp.end());
        int x = n - 2 * sz[v];

        int c = std::lower_bound(tmp.begin(), tmp.end(), x) - tmp.begin();
        ans[x] -= 1ll * c * (c - 1) / 2; // 容斥
    }

    std::sort(G[rt].begin(), G[rt].end(), [&] (int x, int y) -> bool {
        return sz[x] > sz[y];
    });

    if (G[rt].size() > 1) {
        bit[0].init(), bit[1].init();
        id.clear();

        get(G[rt][0], rt, 0);
        for (int i = 1; i < G[rt].size(); i ++) {
            get(G[rt][i], rt, 1);
        }

        std::sort(id.begin(), id.end(), [&] (auto x, auto y) -> bool {
            return a[x.first] < a[y.first];
        });
        for (auto [u, t] : id) {
            ans[a[u]] += bit[t ^ 1].ask(std::max(1, n - a[u] - b[u]));
            bit[t].add(b[u], 1);
        }
    }

    if (G[rt].size() > 2) {
        bit[0].init(), bit[1].init();
        id.clear();

        get(G[rt][1], rt, 0);
        for (int i = 2; i < G[rt].size(); i ++) {
            get(G[rt][i], rt, 1);
        }

        std::sort(id.begin(), id.end(), [&] (auto x, auto y) -> bool {
            return a[x.first] < a[y.first];
        });
        for (auto [u, t] : id) {
            ans[a[u]] += bit[t ^ 1].ask(std::max(1, n - a[u] - b[u]));
            bit[t].add(b[u], 1);
        }
    }

    for (int i = 1; i <= n; i ++) {
        std::cout << ans[i] << " \n"[i == n];
    }
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
5
1 2
2 3
2 4
2 5

1
7
3 4
1 5
2 3
2 6
5 2
7 3
*/
```