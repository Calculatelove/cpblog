---
title: 「2025 ICPC 武汉站」A. Planting Trees
date: 2025-11-07 22:55:44
updated: 2025-11-07 22:55:44
categories: ICPC
tags:
  - 类欧几里德算法
---

# Description

Link：[QOJ 14719](https://qoj.ac/contest/2568/problem/14719)

{% note default %}

有 $T$ 组数据，每组数据给出 $f, x, g, y, n, m$，你需要求出

$$
\sum_{i = 0}^{n - 1} \left[ (fi + x) \bmod m < (gi + y) \bmod m \right]
$$

数据范围：$1 \leq T \leq 10^4$，$1 \leq f, x, g, y, n, m \leq 10^6$。

时空限制：$1$s / $1024$MiB。

{% endnote %}

<!-- more -->

# Solution

一个非负整数对 $m$ 取模以后的值域为 $[0, m - 1]$。对于 $u, v \in [0, m - 1]$，有

$$
[u < v] \iff [u \leq v - 1] \iff 1 + \left\lfloor \frac{v - u - 1}{m} \right\rfloor \quad (u, v \in [0, m - 1])
$$

根据 $a \bmod b = a - \left\lfloor \frac{a}{b} \right\rfloor b$ 展开原式，得

$$
\begin{aligned}
& \sum_{i = 0}^{n - 1} \left[ (fi + x) - \left\lfloor \frac{fi + x}{m} \right\rfloor m < (gi + y) - \left\lfloor \frac{gi + y}{m} \right\rfloor m \right] \\
= & \sum_{i = 0}^{n - 1} \left( 1 + \left\lfloor \frac{(gi + y) - \left\lfloor \frac{gi + y}{m} \right\rfloor m - (fi + x) + \left\lfloor \frac{fi + x}{m} \right\rfloor m - 1 }{m} \right\rfloor \right) \\
= & n + \sum_{i = 0}^{n - 1} \left\lfloor \frac{fi + x}{m} \right\rfloor - \sum_{i = 0}^{n - 1} \left\lfloor \frac{gi + y}{m} \right\rfloor + \sum_{i = 0}^{n - 1} \left\lfloor \frac{(g - f)i + (y - x - 1)}{m} \right\rfloor
\end{aligned}
$$

发现上面的三个求和式，都是类欧的标准问题形式（不过该问题中类欧的参数可能为负数，需要特殊处理）。

时间复杂度 $\mathcal{O}(T \log n)$。

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

/* 类欧几里德算法（要求 a, b >= 0 且 c > 0） */
s64 euclid(s64 a, s64 b, s64 c, s64 n) {
    if (a == 0) {
        return (n + 1) * (b / c);
    }
    if (a >= c || b >= c) {
        return euclid(a % c, b % c, c, n) +
        (n * (n + 1) / 2) * (a / c) + (n + 1) * (b / c);
    }
    s64 m = (a * n + b) / c;
    if (m == 0) {
        return 0;
    }
    return n * m - euclid(c, c - b - 1, a, m - 1);
}

/* 类欧几里德算法 */
s64 euclid_full(s64 a, s64 b, s64 c, s64 n) {
    if (c < 0) {
        a = -a, b = -b, c = -c;
    }
    s64 res = 0;
    if (a < 0) {
        s64 t = (a - c + 1) / c;
        a -= t * c, res += (n * (n + 1) / 2) * t;
    }
    if (b < 0) {
        s64 t = (b - c + 1) / c;
        b -= t * c, res += (n + 1) * t;
    }
    return res + euclid(a, b, c, n);
}

void work() {
    s64 k1, b1, k2, b2, n, m;
    std::cin >> k1 >> b1 >> k2 >> b2 >> n >> m;

    s64 ans = n;
    ans += euclid_full(k1, b1, m, n - 1);
    ans -= euclid_full(k2, b2, m, n - 1);
    ans += euclid_full(k2 - k1, b2 - b1 - 1, m, n - 1);

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

/*
1
7 4 6 3 3 4
*/
```