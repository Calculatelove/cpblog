---
title: 「CF2145G」Cost of Coloring
date: 2025-10-10 16:25:36
updated: 2025-10-10 16:25:36
categories: Codeforces
tags:
  - DP
---

# Description

Link：[CF2145G](https://codeforces.com/contest/2145/problem/G)

{% note default %}

有一个 $n \times m$ 的网格图。最初，每个格点都没有颜色。

每次操作，你可以选择任意一行或一列染色。在第一次操作中，你只能选择颜色 $1$；在第 $i(i > 1)$ 次操作，你可以选择颜色 $c_{i - 1}$ 或 $c_{i - 1} + 1$（其中 $c_{i - 1}$ 表示第 $i - 1$ 次操作选择的颜色）。

如果满足以下条件，我们称最终的染色结果为优美：
- 每个单元格都被染色。
- $1 \sim k$ 的每个颜色都被使用（至少有一个单元格被染成该颜色）。

对于优美的最终染色，我们定义**优美值**为实现该染色方案最少需要的操作次数。

对于 $i = \min(n, m), \dots, n + m - 1$，你需要求出优美值为 $i$ 的优美染色个数。

数据范围：$2 \leq n, m \leq 2000$，$1 \leq k \leq n + m - 1$。

时空限制：$3$s / $512$MiB。

{% endnote %}

<!-- more -->

# Solution

题目中的颜色选取，相当于是每个颜色的染色操作连续。

首先，每行每列至多被染色一次。

考虑颜色 $k$，一定是最后若干次操作染色的，于是颜色 $k$ 必定恰好覆盖了整个网格的若干行与若干列。将这些行与列从网格图上去掉，就变成了一个规模为 $k - 1$ 的子问题。

如此递归直到 $1$。当仅剩颜色 $1$ 的时候，情况又有所不同。设剩下 $x$ 行 $y$ 列，由于我们需要最快染色，我们只需要在“行全染”或“列全染”中选择一种最快的染色方法即可，所以最后可以省下 $\max(x, y)$ 次染色。

考虑枚举颜色 $1$ 所占的行数 $x$ 与列数 $y$，贡献给优美值为 $n + m - \max(x, y)$ 的答案。

行列的选法显然为 $\binom{n}{x}\binom{m}{y}$，此外还剩 $n + m - x - y$ 次染色，需要染上共记 $k - 1$ 个颜色。

设 $f_{i, j}$ 表示共 $i$ 次染色，已经染了 $j$ 个颜色的方案数。有两种转移
- 用以前的颜色：$f_{i, j} \gets_+ f_{i - 1, j} \times j$。
- 新开一个颜色：$f_{i, j} \gets_+ f_{i - 1, j - 1} \times (k - j)$。

则剩下的染色方案为 $f_{n + m - x - y, k - 1}$。

时间复杂度 $\mathcal{O}(n^2)$。

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

const int mod = 998244353; // 模数需根据实际问题调整

/* 模意义下 加法 */
inline void add(int &x, const int &y) {
    x += y;
    if (x >= mod) {
        x -= mod;
    }
}
/* 模意义下 减法 */
inline void dec(int &x, const int &y) {
    x -= y;
    if (x < 0) {
        x += mod;
    }
}
/* 模意义下 取反 */
inline void neg(int &x) {
    if (x) {
        x = mod - x;
    }
}
/* 模意义下 乘法 */
inline void mul(int &x, const int &y) {
    x = 1ll * x * y % mod;
}

/* 模意义下 修正 */
inline int norm(int x) {
    x %= mod;
    return x < 0 ? x + mod : x;
}

/* 快速幂 */
constexpr int qpow(int a, int b, int p) {
    int ans = 1;
    for (; b; b >>= 1) {
        if (b & 1) ans = 1ll * ans * a % p;
        a = 1ll * a * a % p;
    }
    return ans;
}

const int N = 2010;

int n, m, k;

int f[N * 2][N * 2];
int ans[N * 2];

struct BinomCoef {
    std::vector<int> fact, facv;

    void init(const int &n) {
        fact.resize(n + 1), facv.resize(n + 1);
        
        fact[0] = 1;
        for (int i = 1; i <= n; i ++) {
            fact[i] = 1ll * fact[i - 1] * i % mod;
        }

        facv[n] = qpow(fact[n], mod - 2, mod);
        for (int i = n - 1; i >= 0; i --) {
            facv[i] = 1ll * facv[i + 1] * (i + 1) % mod;
        }
    }

    int binom(int n, int m) {
        if (n < m || m < 0) {
            return 0;
        }
        return 1ll * facv[m] * facv[n - m] % mod * fact[n] % mod;
    }
} bc;

void work() {
    std::cin >> n >> m >> k, k --;

    f[0][0] = 1;
    for (int i = 1; i <= n + m; i ++) {
        for (int j = 1; j <= std::min(i, k); j ++) {
            f[i][j] = 0;
            add(f[i][j], 1ll * f[i - 1][j] * j % mod);
            add(f[i][j], 1ll * f[i - 1][j - 1] * (k - j + 1) % mod);
        }
    }

    bc.init(std::max(n, m));

    for (int i = std::min(n, m); i <= n + m - 1; i ++) {
        ans[i] = 0;
    }
    for (int x = 1; x <= n; x ++) {
        for (int y = 1; y <= m; y ++) {
            int cur = 1ll * bc.binom(n, x) * bc.binom(m, y) % mod * f[n + m - x - y][k] % mod;
            add(ans[n + m - std::max(x, y)], cur);
        }
    }

    for (int i = std::min(n, m); i <= n + m - 1; i ++) {
        std::cout << ans[i] << ' ';
    }
    std::cout << '\n';
}

int main() {
    std::ios::sync_with_stdio(0);
    std::cin.tie(0);

    work();

    return 0;
}

/**
 *  心中无女人
 *  比赛自然神
 *  模板第一页
 *  忘掉心上人
**/
```