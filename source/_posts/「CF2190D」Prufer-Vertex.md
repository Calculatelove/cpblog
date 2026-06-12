---
title: 「CF2190D」Prufer Vertex
date: 2026-01-18 17:21:46
updated: 2026-01-18 17:21:46
categories: Codeforces
tags:
  - 「数学」
  - Prufer 序列
---

# Description

Link：[CF2190D](https://codeforces.com/contest/2190/problem/D)

{% note default %}

对于一棵树 $T$，定义 $P(T)$ 表示对 $T$ 生成 Prufer 序列时，最后剩下的两个节点之中不是 $n$ 的那一个。

给出一个包含 $n$ 个点 $m$ 条边的森林。对于每一个 $v$ ($1 \leq v \leq n$)，计算有多少种加边方式，使得原图变成一棵树 $T$，且 $P(T) = v$。答案对 $998244353$ 取模。

数据范围：$2 \leq n \leq 2 \times 10^5$，$0 \leq m \leq n - 1$。

时空限制：$2$s / $256$MiB。

{% endnote %}

<!-- more -->

# Solution

{% note info %}

<span class="highlight">扩展 Cayley 公式</span>：对于一个包含 $n$ 个点 $m$ 条边的森林，设其有 $k$ 个连通块，大小分别为 $s_1, s_2, \dots, s_k$，则可以使得森林变成树的加边方式恰有

$$
n^{k - 2}\prod_{i = 1}^k s_i
$$

<details>
<summary> 证明 </summary>

考虑先将连通块缩成一个点，相当于是在包含 $k$ 个点的新图上进行连边使其成为一棵树。对于一条连接连通块 $x, y$ 的边，一共有 $s_x \times s_y$ 种选法。

设新图连通块 $i$ 的度数为 $d_i$，则在新图连边确定的情况下，方案数为 $\prod_{i = 1}^k s_i^{d_i}$。

回顾一下 Prufer 序列的性质：每个节点在 Prufer 序列中的出现次数，是其度数减 $1$。

于是，对于新图的 Prufer 序列而言，连通块 $i$ 的出现次数 $c_i = d_i - 1$。

$$
\begin{aligned}
\sum \prod_{i = 1}^k s_i^{d_i} & = \left( \prod_{i = 1}^k s_i \right) \cdot \sum \prod_{i = 1}^k s_i^{d_i - 1} \\
& = \left( \prod_{i = 1}^k s_i \right) \cdot \sum \prod_{i = 1}^k s_i^{c_i} \\
& = \left( \prod_{i = 1}^k s_i \right) \cdot (s_1 + \dots + s_k)^{k - 2} \\
& = n^{k - 2} \prod_{i = 1}^k s_i
\end{aligned}
$$

相当于是先将贡献 $\prod_{i = 1}^k s_i$ 取出。然后对于一种 Prufer 序列，连通块 $i$ 每出现一次，就会有乘以 $s_i$ 的贡献。

由于 Prufer 序列上每个位置是独立的，一个位置共有 $s_1 + \dots + s_k = n$ 种方案，则 $k - 2$ 个位置共有 $n^{k - 2}$ 种方案。故总方案数为 $n^{k - 2}\prod_{i = 1}^k s_i$。

</details>

{% endnote %}

先思考一下 $P(T)$ 的值是什么。注意到 $n - 1$ 在全局仍有除了 $n - 1$ 以外的其他叶子时，不会被删除。换言之，当且仅当全局只有 $n - 1$ 一个叶子时，$n - 1$ 会被删除。**此时全局一定是形成了一条从 $n - 1$ 到 $n$ 的链**。那么后续的操作就是从链的 $n - 1$ 端开始删，直到只剩两个点时。

**故 $P(T)$ 是从 $n$ 到 $n - 1$ 路径上的第二个点**。

我们以 $n$ 为根，方便统计答案。记 $\mathrm{tot} = n^{k - 2}\prod_{i = 1}^k s_i$。

当 $n$ 与 $n - 1$ 已经在同一个连通块时，此时 $P(T) = v$ 固定，令 $\mathrm{ans}_v \gets \mathrm{tot}$。

否则，我们先考虑已经和 $n$ 直接相连的点 $v$（也就是 $n$ 的儿子 $v$）。如果先切断 $n$ 所在连通块与外界的连接，我们就是要 $n - 1$ 所在的部分直接连向 $v$ 所在的子树。

**若外界想要连向 $n$ 所在连通块的某一点时，连接每个点的概率都是一样的（即比例一样，均占总数的 $\frac{1}{\mathrm{size}_n}$）**。所以令 $\mathrm{ans}_v \gets \frac{\mathrm{size}_v}{\mathrm{size}_n} \mathrm{tot}$。

对于其余连通块的点 $v$，首先我们需要先连一条从 $n$ 到 $v$ 的边。然后相当于是对剩下的 $k - 1$ 个连通块，进行同上的讨论。再次套用扩展 Cayley 公式计算即可。

时间复杂度 $\mathcal{O}(n)$ 或 $\mathcal{O}(n \log n)$。

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

const int mod = 998244353; // 模数需要根据实际问题调整

// 模意义下 修正
template <class T>
inline int norm(T x) {
    x %= mod;
    return x < 0 ? x + mod : x;
}

// 模意义下 加法
inline void add(int &x, const int &y) {
    x += y;
    if (x >= mod) {
        x -= mod;
    }
}
// 模意义下 减法
inline void dec(int &x, const int &y) {
    x -= y;
    if (x < 0) {
        x += mod;
    }
}
// 模意义下 取反
inline void neg(int &x) {
    if (x) {
        x = mod - x;
    }
}
// 模意义下 乘法
inline void mul(int &x, const int &y) {
    x = 1ll * x * y % mod;
}

// 快速幂
constexpr int qpow(int a, int b, int p) {
    int ans = 1;
    for (; b; b >>= 1) {
        if (b & 1) ans = 1ll * ans * a % p;
        a = 1ll * a * a % p;
    }
    return ans;
}

const int N = 200100;

int n, m;
std::vector<std::vector<int>> G;

int ans[N];

int vis[N];
std::vector<int> seq;

int Fa[N], sze[N];
void dfs_init(int u, int fu) {
    vis[u] = 1;
    Fa[u] = fu, sze[u] = 1;
    for (int v : G[u]) {
        if (v == fu) {
            continue;
        }
        dfs_init(v, u);
        sze[u] += sze[v];
    }
}

void assign(int u, int fu, int val) {
    ans[u] = val;
    for (int v : G[u]) {
        if (v == fu) {
            continue;
        }
        assign(v, u, val);
    }
}

void work() {
    std::cin >> n >> m;

    G.assign(n + 1, {});
    for (int i = 1; i <= m; i ++) {
        int x, y;
        std::cin >> x >> y;
        G[x].push_back(y), G[y].push_back(x);
    }

    std::fill(ans + 1, ans + 1 + n, 0);
    std::fill(vis + 1, vis + 1 + n, 0);
    seq.clear();

    for (int i = n; i >= 1; i --) {
        if (vis[i]) {
            continue;
        }
        seq.push_back(i);
        dfs_init(i, 0);
    }

    int tot = 1;
    if (seq.size() > 1) {
        mul(tot, qpow(n, seq.size() - 2, mod));
        for (int rt : seq) {
            mul(tot, sze[rt]);
        }
    }

    {
        int p = n - 1;
        while (p && Fa[p] != n) {
            p = Fa[p];
        }

        if (p) {
            ans[p] = tot;
            for (int i = 1; i < n; i ++) {
                std::cout << ans[i] << ' ';
            }
            std::cout << '\n';
            return;
        }
    }

    for (int v : G[n]) {
        ans[v] = 1ll * tot * qpow(sze[n], mod - 2, mod) % mod * sze[v] % mod;
    }
    for (int rt : seq) {
        if (rt == n) {
            continue;
        }
        if (rt == n - 1) {
            int val = 1ll * tot * qpow(1ll * n * sze[n] % mod * sze[rt] % mod, mod - 2, mod) % mod * (sze[n] + sze[rt]) % mod;
            assign(rt, 0, val);
        } else {
            // int val = 1ll * tot * qpow(1ll * n * sze[n] % mod * sze[rt] % mod, mod - 2, mod) % mod * sze[rt] % mod;
            int val = 1ll * tot * qpow(1ll * n * sze[n] % mod, mod - 2, mod) % mod;
            assign(rt, 0, val);
        }
    }

    for (int i = 1; i < n; i ++) {
        std::cout << ans[i] << ' ';
    }
    std::cout << '\n';
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
5 3
1 5
3 5
4 5

1
4 1
1 2
*/
```