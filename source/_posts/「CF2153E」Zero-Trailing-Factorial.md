---
title: 「CF2153E」Zero Trailing Factorial
date: 2025-10-12 20:53:45
updated: 2025-10-12 20:53:45
categories: Codeforces
tags:
  - 「数学」
  - 约数
---

# Description

Link：[CF2153E](https://codeforces.com/contest/2153/problem/E)

{% note default %}

设 $v_p(x!)$ 表示 $x!$ 可以分解出多少个 $p$ 的乘积。

勒让德定理：
- 当 $p$ 为质数时
$$
v_p(x!) = \sum_{j = 1}^{\lfloor \log_k x \rfloor} \left\lfloor \frac{x}{p^j} \right\rfloor
$$
- 当 $p$ 为合数时，记 $p = \sum_{i = 1}^m p_i^{c_i}$
$$
v_p(x!) = \min_{i = 1}^m \left\lfloor \frac{v_{p_i}(x!)}{c_i} \right\rfloor
$$

设

$$
w_k(a, b) = \begin{cases}
\min(v_k(a!), v_k(b!)) & \text{if }v_k(a!) \neq v_k(b!) \text{;}\\
10^{100} & \text{otherwise.}
\end{cases}
$$

设

$$
f_m(a, b) = \min_{2 \leq k \leq m} w_k(a, b)
$$

给出两个整数 $n, m$，你需要计算 $\sum\limits_{1 \leq x < n}f_m(x, n)$。

数据范围：$2 \leq n \leq m \leq 10^7$。可以证明给定条件下答案严格小于 $10^{100}$。

时空限制：$3$s / $512$MiB。

{% endnote %}

<!-- more -->

# Solution

设 $p_0$ 是小于等于 $n$ 中最大的质数，对于 $x < p_0$ 显然有 $f_m(x, n) = 0$（因为取 $k = p_0$ 即有 $w_k(x, n) = 0$）。

**打表发现 $10^7$ 以内，相邻素数的差值最大为 $154$**。于是我们真正要考虑的 $p_0 \leq x < n$ 部分，实际上并不大。

{% note info %}

**性质**：对于 $a \leq b$，有 $v_k(a!) \leq v_k(b!)$。

由于 $a! \mid b!$，易证。

{% endnote %}

{% note info %}

**性质**：$w_k(a, b) \geq \min_{i = 1}^m w_{p_i^{c_i}}(a, b)$。

对于 $1 \leq i \leq m$，设 $a_i = \left\lfloor \frac{v_{p_i}(a!)}{c_i} \right\rfloor$，$b_i = \left\lfloor \frac{v_{p_i}(b!)}{c_i} \right\rfloor$。
- 若 $v_k(a!) = v_k(b!)$，则 $w_k(a, b) = 10^{100}$，不等式成立。
- 若 $v_k(a!) \neq v_k(b!)$，即 $\min\{a_i\} < \min\{b_i\}$，设 $a_x = \min\{ a_i \}$，则 $a_x < b_x$，此时有 $w_k(a, b) = a_x = w_{p_x^{c_x}}(a, b)$。

该性质告诉我们，我们只需枚举形如 $k = p^c$ 的数计算 $f_m(x, n)$ 即可。

{% endnote %}

{% note info %}

**性质**：只需枚举形如 $k = p^c$ 且 $p$ 被 $[p_0, n]$ 中至少一个数整除的数。

记 $n! = p_0!(p_0 + 1) \dots n$，若 $p$ 不被 $[p_0, n]$ 中的某个数整除，则必有 $v_k(p_0!) = v_k(n!)$。

{% endnote %}

{% note info %}

**性质**：答案必定严格小于 $10^{100}$。

考虑将 $x - 1$ 变成 $x$，对于 $x$ 的某个质因数 $p$，必有 $v_p((x - 1)!) < v_{p}(x!)$，因为 $\left\lfloor \frac{x}{p} \right\rfloor$ 发生了变化。

{% endnote %}

于是将 $[p_0, n]$ 中所有数的质因数拿出来，枚举形如 $k = p^c$ 的数计算 $f_m(x, n)$ 即可。

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

const int V = 1e7, MaxV = V + 10;

int primeCount, prime[MaxV], fac[MaxV];
int low[MaxV];

void sieve(const int &n) {
    for (int i = 2; i <= n; i ++) {
        if (!fac[i]) {
            prime[++ primeCount] = i, fac[i] = i;
            low[i] = i;
        }
        for (int j = 1; j <= primeCount; j ++) {
            if (prime[j] > fac[i] || prime[j] > n / i) break;
            fac[i * prime[j]] = prime[j];
            low[i * prime[j]] = prime[j] * (i % prime[j] == 0 ? low[i] : 1);
        }
    }
}

int n, m;

int factorialIndex(int n, int p) {
    int cnt = 0;
    for (int v = p;; v *= p) {
        cnt += n / v;

        if (v > n / p) {
            break;
        }
    }
    return cnt;
}

void work() {
    std::cin >> n >> m;

    int p0 = n;
    while (fac[p0] != p0) {
        p0 --;
    }

    std::vector<int> key;

    for (int i = p0; i <= n; i ++) {
        int x = i;
        while (x > 1) {
            key.push_back(fac[x]);
            x /= low[x];
        }
    }

    std::sort(key.begin(), key.end());
    key.erase(std::unique(key.begin(), key.end()), key.end());

    s64 ans = 0;
    for (int i = p0; i < n; i ++) {
        int cur = 0x3f3f3f3f;
        for (int p : key) {
            int cnti = factorialIndex(i, p),
                cntn = factorialIndex(n, p);
            for (int v = p, c = 1;; v *= p, c ++) {
                int vi = cnti / c,
                    vn = cntn / c;
                if (vi < vn) {
                    chmin(cur, vi);
                }

                if (v > m / p) {
                    break;
                }
            }
        }
        ans += cur;
    }

    std::cout << ans << '\n';
}

int main() {
    std::ios::sync_with_stdio(0);
    std::cin.tie(0);

    sieve(V);

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