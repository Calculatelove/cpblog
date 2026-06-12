---
title: 「CF2150D」Attraction Theory
date: 2025-09-26 15:33:59
updated: 2025-09-26 15:33:59
categories: Codeforces
tags:
  - 「数学」
  - 组合数学
  - 期望
---

# Description

Link：[CF2150D](https://codeforces.com/contest/2150/problem/D)

{% note default %}

有 $n$ 个人，分别位于坐标轴上的 $p_1, \dots, p_n$ 位置。初始时 $p_i = i$。

你可以在坐标轴上的某个位置 $x(1 \leq x \leq n)$ 放置一个景点，接下来所有人都会朝着这个景点靠拢。具体地，对于每个人 $i$：
- 若 $p_i = x$，则位置不变。
- 若 $p_i < x$，则向右走一步 $p_i \gets p_i + 1$。
- 若 $p_i > x$，则向左走一步 $p_i \gets p_i - 1$。

每个位置 $x$ 都有一个对应的权值 $a_x$，对于某一时刻的站位数组 $p_1, \dots, p_n$，得分定义为
$$
\sum_{i = 1}^n a_{p_i}
$$

你可以进行任意次操作，每次放置景点的位置任选。对于所有可能的站位数组 $p$，你需要求出得分总和。答案对 $998244353$ 取模。

数据范围：$1 \leq n \leq 2 \times 10^5$，$1 \leq a_i \leq 10^9$。

时空限制：$2$s / $256$MiB。

{% endnote %}

<!-- more -->

# Solution

先考虑最后的站位数组 $p$ 的形状。所有人的位置应该构成连续的一段区间，每次操作 $x$ 时，若 $x$ 在区间内部，则相当于是将 $x - 1, x, x + 1$ 位置上的所有人都缩成一个点。$x$ 在区间边缘的话，则比较特殊，相当于是只影响了两个位置。

通过归纳，可以得出一个**必要条件**：
- 所有人的位置构成连续的一段区间。
- 区间边缘的人数随意，区间内部的人数必为奇数，区间人数总和为 $n$。

同时我们发现，**该条件也是充分的**，因为任意一个满足该必要条件的位置数组 $p$ 都可以被构造出来：从左往右构造。若边缘位置人数为 $v$，则只需要放置 $v - 1$ 次景点；若中间位置人数为 $v$，则只需要放置 $\frac{v - 1}{2}$ 次景点。等整个位置数组 $p$ 的形状构造出来以后，可以通过在 $1$ 或 $n$ 处放置景点，从而调整区间的起始位置。

假设最后的区间长度是 $k$，不妨设区间每个位置的人数依次是 $f_1, \dots, f_k$。对于 $1 < i < k$，$f_i$ 是奇数。而 $f_1, f_k$ 的奇偶性任意。

考虑钦定 $f_1, f_k$ 的奇偶性，用 $x, y(1 \leq x, y \leq 2)$ 来刻画奇偶性。记
- $f_1 = 2g_1 + x$，$f_k = 2g_k + y$。
- $f_i = 2g_i + 1$（其中 $1 < i < n$）。

记 $S = n - (k - 2) - x - y$，则有 $\sum 2g_i = S$（此时的 $S$ 必须为偶数），**由于所有 $g_i$ 的地位都是相同的，故所有 $g_i$ 的期望值均为 $\frac{S}{2k}$**。我们还需要计算一共有多少种数组 $\{g_i\}$，由插板法得 $\binom{S + k - 1}{k - 1}$。

**由期望值乘以方案数即可得到总和**。故此时的贡献为 $(\frac{S}{k} + 1) \cdot \binom{S + k - 1}{k - 1} \cdot \mathrm{sum}_k$，其中 $\mathrm{sum}_k$ 表示所有长度为 $k$ 的区间的总和，可以对前缀和再次前缀和得到。特别地要考虑 $x = 2$ 以及 $y = 2$ 时的贡献。

时间复杂度 $\mathcal{O}(n)$。

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

/* 模意义下 乘法 */
inline void mul(int &x, const int &y) {
    x = 1ll * x * y % mod;
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

const int N = 200100;

int n;
int a[N];

int pre[N], pre2[N];

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
    std::cin >> n;

    for (int i = 1; i <= n; i ++) {
        std::cin >> a[i];
        a[i] %= mod;
    }

    bc.init(n * 2);

    for (int i = 1; i <= n; i ++) {
        add(pre[i] = pre[i - 1], a[i]);
        add(pre2[i] = pre2[i - 1], pre[i]);
    }

    int ans = 1ll * n * pre[n] % mod;

    for (int k = 2; k <= n; k ++) {
        int sum = 0;
        add(sum, pre2[n]), dec(sum, pre2[k - 1]);
        dec(sum, pre2[n - k]);

        // std::cout << sum << '\n';

        for (int x = 1; x <= 2; x ++) {
            for (int y = 1; y <= 2; y ++) {
                int S = n - x - y - (k - 2);
                if (S & 1) {
                    continue;
                }

                S >>= 1;

                int val = 1ll * S * qpow(k, mod - 2, mod) % mod;
                int cnt = bc.binom(S + k - 1, k - 1);

                add(ans, (2ll * val + 1) * cnt % mod * sum % mod);
                if (x > 1) {
                    add(ans, 1ll * cnt * pre[n - k + 1] % mod);
                }
                if (y > 1) {
                    add(ans, 1ll * cnt * (pre[n] - pre[k - 1] + mod) % mod);
                }
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

/**
 *  心中无女人
 *  比赛自然神
 *  模板第一页
 *  忘掉心上人
**/
```