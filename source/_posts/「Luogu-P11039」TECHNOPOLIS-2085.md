---
title: 「Luogu P11039」TECHNOPOLIS 2085
date: 2026-01-18 20:04:56
updated: 2026-01-18 20:04:56
categories: Luogu
tags:
  - 组合计数
---

# Description

Link：[Luogu P11039](https://www.luogu.com.cn/problem/P11039)

{% note default %}

给出一棵包含 $n$ 个点的有根树 $T$，根节点为 $1$。给出一个大小为 $m$ 的关键点集合 $S$。

你需要求出有多少个不同的**有根树** $T'$，满足对于任意 $x, y \in S$，均有 $\mathrm{LCA}_T(x, y) = \mathrm{LCA}_{T'}(x, y)$。

答案对 $998244353$ 取模。

数据范围：$2 \leq m \leq n \leq 10^6$。

时空限制：$1$s / $512$MiB。

{% endnote %}

<!-- more -->

# Solution

首先得出集合 $S$ 的虚树大小 $r$。

假设 $T'$ 以集合 $S$ 的虚树树根为根，记这样的 $T'$ 数量为 $f(n, r)$。

考虑如何计算 $f(n, r)$。枚举 $i$ ($0 \leq i \leq n - r$) 表示在剩下的 $n - r$ 个点中，有多少个点插入了虚树的内部。其余点都在虚树的外部。

插入虚树内部的方案数为 $\binom{n - r}{i}(r - 1)^{\overline{i}}$。此时还剩下 $n - r - i$ 个孤立点，和一个大小为 $r + i$ 的连通块。由扩展 Cayley 公式可知，加边方式共有 $n^{n - r - i - 1}(r + i)$ 种。

故

$$
f(n, r) = \sum_{i = 0}^{n - r} \binom{n - r}{i}(r - 1)^{\overline{i}} n^{n - r - i - 1}(r + i)
$$

此外，当 $T'$ 的根为其他节点时。我们可以从剩下的 $n - r$ 个点中，任选一个点出来做根。此时我们可以认为虚树里包含这个根，就转化成了一个大小为 $r + 1$ 的虚树。

综上，答案为

$$
f(n, r) + (n - r) f(n, r + 1)
$$

时间复杂度 $\mathcal{O}(n)$。

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

const int N = 1000100;

int n, m, r;

std::vector<std::vector<int>> G;

int sum[N];
int iskey[N];

void dfs(int u) {
    int s = 0;
    sum[u] = iskey[u];
    for (int v : G[u]) {
        dfs(v);
        s += sum[v] > 0;
        sum[u] += sum[v];
    }

    if (iskey[u] || s > 1) {
        r ++;
    }
}

// 组合数（预处理阶乘与阶乘逆元）
struct BinomCoef {
    std::vector<int> fact, facv;

    BinomCoef() {}
    BinomCoef(int n) {
        init(n);
    }

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

int f(int n, int r) {
    if (r > n) return 0;
    if (r == n) return 1;

    int in = qpow(n, mod - 2, mod);
    int w = qpow(n, n - r - 1, mod);
    int v = 1;

    int ans = 0;
    for (int i = 0; i <= n - r; i ++) {
        ans = (ans + 1ll * bc.binom(n - r, i) * v % mod * w % mod * (r + i)) % mod;

        mul(w, in);
        mul(v, r - 1 + i);
    }

    return ans;
}

int main() {
    std::ios::sync_with_stdio(0);
    std::cin.tie(0);

    std::cin >> n >> m;

    G.assign(n + 1, {});
    for (int i = 2; i <= n; i ++) {
        int x;
        std::cin >> x;
        G[x].push_back(i);
    }

    for (int i = 1; i <= m; i ++) {
        int x;
        std::cin >> x;
        iskey[x] = 1;
    }

    dfs(1);

    bc.init(n);

    int ans = 0;
    add(ans, f(n, r));
    add(ans, 1ll * (n - r) * f(n, r + 1) % mod);

    std::cout << ans << '\n';

    return 0;
}
```