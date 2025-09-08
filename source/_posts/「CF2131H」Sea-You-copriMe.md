---
title: 「CF2131H」Sea, You & copriMe
date: 2025-09-08 10:01:00
updated: 2025-09-08 10:01:00
categories: Codeforces
tags:
  - 莫比乌斯反演
---

# Description

Link：[CF2131H](https://codeforces.com/problemset/problem/2131/H)

{% note default %}

给出一个长度为 $n$ 的数组 $a$，其中数组中的元素均为 $1 \sim m$ 中的正整数。

你需要找到四个不同的索引 $p, q, r, s$，使得 $\gcd(a_p, a_q) = 1$ 且 $\gcd(a_r, a_s) = 1$。

数据范围：$4 \leq n \leq 2 \times 10^5$，$1 \leq m \leq 10^6$，$1 \leq a_i \leq m$。

时空限制：$3$s / $256$MiB。

{% endnote %}

<!-- more -->

# Solution

考虑对于一个数 $x$，判断序列中是否存在与 $x$ 互质的数，设 $\mathrm{cnt}_i$ 表示 $i$ 的出现次数。相当于是要求

$$
\begin{aligned}
& \sum\limits_{i = 1}^m \mathrm{cnt}_i [\gcd(i, x) = 1] \\
& = \sum\limits_{i = 1}^m \mathrm{cnt}_i \sum\limits_{d \mid i, d \mid x} \mu(d) \\
& = \sum\limits_{d \mid x} \mu(d) \sum\limits_{d \mid i} \mathrm{cnt}_i
\end{aligned}
$$

首先可以求出一对 $p, q$。从左向右遍历，假设当前遍历到 $i$，首先 $\mathcal{O}(\sqrt{m})$ 地求出上式的值，若不为 $0$，则说明 $1 \sim i - 1$ 必定有一个与 $a_i$ 互质的数，暴力找出即可。否则，$\mathcal{O}(\sqrt{m})$ 地更新 $\sum_{d \mid i} \mathrm{cnt}_i$ 的值。

然后将 $p, q$ 从序列中拿出来，继续尝试找出一对 $r, s$（进行重复的事情）。

若此时找不到 $r, s$，那就有可能是因为 $p, q$ 绑定在了一起，导致其他数不能与 $p, q$ 进行匹配。所以剩余的一种情况是，一个数与 $p$ 匹配，还有一个数与 $q$ 匹配。遍历剩下的数，尝试匹配即可。

时间复杂度 $\mathcal{O}(n\sqrt{m})$。

```c++
#include <bits/stdc++.h>

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

const int V = 1e6;
const int MaxV = V + 10;

int primeCount, prime[MaxV], fac[MaxV], miu[MaxV];
std::vector<int> factor[MaxV];
void sieve(const int &n) {
    miu[1] = 1;
    for (int i = 2; i <= n; i ++) {
        if (!fac[i]) {
            prime[++ primeCount] = i, fac[i] = i, miu[i] = -1;
        }
        for (int j = 1; j <= primeCount; j ++) {
            if (prime[j] > fac[i] || prime[j] > n / i) break;
            fac[i * prime[j]] = prime[j];
            miu[i * prime[j]] = miu[i] * (i % prime[j] ? -1 : 0);
        }
    }

    for (int i = 1; i <= n; i ++) {
        for (int j = i; j <= n; j += i) {
            factor[j].push_back(i);
        }
    }
}

const int N = 200100, M = 1000100;

int n, m;
int a[N], vis[N];

int sum[M];

std::pair<int, int> find() {
    for (int i = 1; i <= m; i ++) {
        sum[i] = 0;
    }
    for (int i = 1; i <= n; i ++) {
        if (vis[i]) {
            continue;
        }
        int b = 0;
        for (int d : factor[a[i]]) {
            b += miu[d] * sum[d], sum[d] ++;
        }
        if (b) {
            for (int j = 1; j < i; j ++) {
                if (vis[j]) {
                    continue;
                }
                if (std::gcd(a[j], a[i]) == 1) {
                    vis[j] = vis[i] = 1;
                    return {j, i};
                }
            }
        }
    }

    return {-1, -1};
}

void work() {
    std::cin >> n >> m;

    for (int i = 1; i <= n; i ++) {
        std::cin >> a[i], vis[i] = 0;
    }

    auto [p1, q1] = find();
    if (p1 == -1) {
        std::cout << 0 << '\n';
        return;
    }

    auto [p2, q2] = find();
    if (p2 != -1) {
        std::cout << p1 << ' ' << q1 << ' ' << p2 << ' ' << q2 << '\n';
        return;
    }

    int p = 0, q = 0;
    for (int i = 1; i <= n; i ++) {
        if (vis[i]) {
            continue;
        }
        int fp = std::gcd(a[i], a[p1]) == 1,
            fq = std::gcd(a[i], a[q1]) == 1;

        if (fp && q) {
            std::cout << i << ' ' << p1 << ' ' << q << ' ' << q1 << '\n';
            return;
        }
        if (fq && p) {
            std::cout << i << ' ' << q1 << ' ' << p << ' ' << p1 << '\n';
            return;
        }

        if (fp) p = i;
        if (fq) q = i;
    }

    std::cout << 0 << '\n';
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