---
title: 「2025 ICPC 网络赛 2」K. The Only Heart
date: 2025-09-14 23:59:26
updated: 2025-09-14 23:59:26
categories: 网络预选赛
tags:
  - DP
  - 树上背包
---

# Description

Link：[QOJ 14324](https://qoj.ac/contest/2524/problem/14324)

{% note default %}

给出一个包含 $n$ 个点的树。

你可以进行若干次操作，每次你可以删除这棵树的一个叶子（即保证剩下部分仍然是连通的）。

你需要求出有多少种删除叶子的方案，使得剩下部分的重心唯一（**这里的重心定义与平常相同，即最大子树最小的节点**）。

答案对 $998244353$ 取模，两个方案不同当且仅当删除叶子节点构成的集合不同。

数据范围：$1 \leq n \leq 3 \times 10^3$。

时空限制：$3$s / $1024$MiB。

{% endnote %}

<!-- more -->

# Solution

{% note info %}

一棵树至多有两棵重心。

若一棵树有两个重心，则这两个重心相邻，并且整棵树可以被相连的这条边，划分成大小相等的两支。

{% endnote %}

有了这个结论就好算了。使用总的连通块个数，减去包含两个重心的连通块个数即可。

#### 总的连通块个数

假定以 $1$ 为根。设 $f(u)$ 表示在 $u$ 的子树内，包含点 $u$ 的连通块个数，则有

$$
f(u) = \prod\limits_{v \in \mathrm{son}(u)} (f(v) + 1)
$$

最后总的连通块个数即为 $\sum_{i = 1}^{n} f(i)$。

#### 包含两个重心的连通块个数

由先前的结论，相当于是要找出有多少个，两个大小相同并且相连的连通块。

记相连的边为**关键边**。这两个连通块在树上的位置关系一定是**一上一下**，并且由关键边相连。我们不妨把下面部分的节点权值设成 $-1$，把上面部分的节点权值设成 $-1$。那么两个部分大小相同，等价于权值和为 $0$。

考虑 dp。设 $f(u, i, 0 / 1)$ 表示在 $u$ 的子树内，包含点 $u$ 的连通块大小为 $i$，是否存在关键边的方案数。当合并两棵子树 $u, v$ 时，有转移

$$
f'(u, i + j, 0) \gets_+ f(u, i, 0) \times f(v, j, 0) \\
f'(u, i + j, 1) \gets_+ f(u, i, 1) \times f(v, j, 0) \\
f'(u, i + j, 1) \gets_+ f(u, i, 0) \times f(v, j, 1) \\
f'(u, i - j, 1) \gets_+ f(u, i, 0) \times f(v, j, 0)
$$

当然这里的 $i, j$ 分别只需要枚举到 $[-\mathrm{sze}_u, \mathrm{sze}_u]$ 与 $[-\mathrm{sze}_v, \mathrm{sze}_v]$ 即可，时间复杂度 $\mathcal{O}(n^2)$。

{% note danger %}

换根树上背包的时间复杂度是 $\mathcal{O}(n^3)$ 的。

{% endnote %}

```c++
#include <bits/stdc++.h>

using s64 = long long;

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

const int mod = 998244353;

void add(int &x, const int &y) {
    x += y;
    if (x >= mod) {
        x -= mod;
    }
}

void dec(int &x, const int &y) {
    x -= y;
    if (x < 0) {
        x += mod;
    }
}

const int N = 3010;

int n;

struct Graph {
    std::vector< std::vector<int> > table;

    void init(int _n) {
        table.assign(_n + 1, {});
    }

    void add_edge(int u, int v) {
        table[u].push_back(v);
    }
} G;

int ans;

int sze[N];

namespace p1 {
    int f[N];

    void dp(int u, int fu) {
        f[u] = 1;
        for (int v : G.table[u]) {
            if (v == fu) continue;
            dp(v, u);
            f[u] = 1ll * f[u] * (f[v] + 1) % mod;
        }
        add(ans, f[u]);
    }
}

namespace p2 {
    int f[N][N * 2][2], tmp[N * 2][2];

    void dp(int u, int fu) {
        sze[u] = 1;
        f[u][0 + n][0] = 0, f[u][1 + n][0] = 1, f[u][-1 + n][0] = 0;
        f[u][0 + n][1] = 0, f[u][1 + n][1] = 0, f[u][-1 + n][1] = 0;

        for (int v : G.table[u]) {
            if (v == fu) continue;

            dp(v, u);

            int t = sze[u] + sze[v];

            for (int i = -t; i <= t; i ++) {
                tmp[i + n][0] = 0;
                tmp[i + n][1] = 0;
            }

            for (int i = -sze[u]; i <= sze[u]; i ++) {
                add(tmp[i + n][0], f[u][i + n][0]);
                add(tmp[i + n][1], f[u][i + n][1]);

                for (int j = -sze[v]; j <= sze[v]; j ++) {
                    add(tmp[i + j + n][0], 1ll * f[u][i + n][0] * f[v][j + n][0] % mod);
                    add(tmp[i + j + n][1], 1ll * f[u][i + n][1] * f[v][j + n][0] % mod);
                    add(tmp[i + j + n][1], 1ll * f[u][i + n][0] * f[v][j + n][1] % mod);
                    add(tmp[i - j + n][1], 1ll * f[u][i + n][0] * f[v][j + n][0] % mod);
                }
            }

            for (int i = -t; i <= t; i ++) {
                f[u][i + n][0] = tmp[i + n][0];
                f[u][i + n][1] = tmp[i + n][1];
            }

            sze[u] = t;
        }

        dec(ans, f[u][0 + n][1]);
    }
}

void work() {
    std::cin >> n;

    G.init(n);
    for (int i = 1; i < n; i ++) {
        int x, y;
        std::cin >> x >> y;

        G.add_edge(x, y), G.add_edge(y, x);
    }

    ans = 0;
    p1::dp(1, 0); // std::cout << ans << '\n';
    p2::dp(1, 0);

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
6
1 2
2 3
2 4
3 5
4 6
*/
```