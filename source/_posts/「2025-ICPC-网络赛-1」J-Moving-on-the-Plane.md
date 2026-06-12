---
title: 「2025 ICPC 网络赛 1」J. Moving on the Plane
date: 2025-09-07 22:54:01
updated: 2025-09-07 22:54:01
categories: ICPC
tags:
  - 「数学」
  - 曼哈顿距离与切比雪夫距离的转化
  - 组合计数
---

# Description

Link：[QOJ 14310](https://qoj.ac/contest/2513/problem/14310)

{% note default %}

有 $n$ 个点，第 $i$ 个点的初始坐标为整点 $(x_i, y_i)$。

接下来有 $m$ 轮移动，每一轮中的所有 $n$ 个点 $(x, y)$ 都需要移动到 $(x + 1, y), (x - 1, y), (x, y + 1), (x, y - 1)$ 四个位置的其中一个。

你需要求出有多少种不同的移动点的方式，使得最终 $n$ 个点两两曼哈顿距离小于等于 $k$。其中 $(x_1, y_1), (x_2, y_2)$ 的曼哈顿距离为 $|x_1 - x_2| + |y_1 - y_2|$。

数据范围：$1 \leq n \leq 50$，$1 \leq m \leq 10^5$，$0 \leq k \leq 10$，$0 \leq |x_i|, |y_i| \leq 10^5$。

时空限制：$2$s / $1024$MiB。

{% endnote %}

<!-- more -->

# Solution

{% note info %}

reference: https://oi.wiki/geometry/distance/

曼哈顿距离与切比雪夫距离的转化：
- 曼哈顿距离 $\to$ 切比雪夫距离：$(x, y) \to (x + y, x - y)$。
- 切比雪夫距离 $\to$ 曼哈顿距离：$(x, y) \to \left( \frac{x + y}{2}, \frac{x - y}{2} \right)$。

证明：只证明第一条。

$$
\begin{aligned}
& |x_1 - x_2| + |y_1 - y_2| \\
= & \max\left\{ x_1 - x_2 + y_1 - y_2, -x_1 + x_2 + y_1 - y_2, x_1 - x_2 - y_1 + y_2, -x_1 + x_2 - y_1 + y_2 \right\} \\
= & \max\left\{ |(x_1 + y_1) - (x_2 + y_2)|, |(x_1 - y_1) - (x_2 - y_2)| \right\}
\end{aligned}
$$

注：式子第二行的第一项与第四项为相反数，第二项与第三项为相反数。

{% endnote %}

考虑曼哈顿距离转切比雪夫距离。

$n$ 个点两两切比雪夫距离 $\leq k$，相当于**两两横坐标之差 $\leq k$ 且纵坐标之差 $\leq k$**。

一个点 $(x, y)$ 的一次移动，相当于移动到 $(x + 1, y + 1), (x + 1, y - 1), (x - 1, y + 1), (x - 1, y - 1)$ 四个位置的其中一个，相当于**横坐标 $\pm 1$ 与纵坐标 $\pm 1$**。

仔细思考可以发现 **$x, y$ 是独立的**！于是我们只需要考虑一维的情况下怎么做即可。

考虑枚举最小值 $l$，则相当于是所有点都要到达 $[l, l + k]$ 并且至少有一个点到达 $l$。可以使用所有点到达 $[l, l + k]$ 的方案数，减去所有点到达 $[l + 1, l + k]$ 的方案数。一个点经过 $m$ 次移动向前 $x$ 步的方案数显然为 $[i \equiv m \pmod 2]\binom{m}{\frac{i + m}{2}}$，使用前缀和优化即可。

时间复杂度 $\mathcal{O}(nm)$。

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

const int N = 100100;

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
int qpow(int a, int b, int p) {
    int ans = 1;
    for (; b; b >>= 1) {
        if (b & 1) ans = 1ll * ans * a % p;
        a = 1ll * a * a % p;
    }
    return ans;
}

const int V = 3e5, MaxV = V * 2 + 10;

int n, m, k;

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

int pre[MaxV];

int walk(int x, int l, int r) {
    if (x <= l) {
        int s = l - x, e = r - x;
        int sum = pre[e];
        if (s) dec(sum, pre[s - 1]);
        return sum;
    } else if (x >= r) {
        int s = x - r, e = x - l;
        int sum = pre[e];
        if (s) dec(sum, pre[s - 1]);
        return sum;
    } else {
        int sum = 0;
        add(sum, pre[r - x]), add(sum, pre[x - l]), dec(sum, pre[0]);
        return sum;
    }
}

int calc(std::vector<int> &a) {
    int mi = *std::min_element(a.begin() + 1, a.end()),
        ma = *std::max_element(a.begin() + 1, a.end());
    int le = std::max(-V, ma - m - k), rg = std::min(V, mi + m);
    int ans = 0;
    for (int l = le; l <= rg; l ++) {
        int v1 = 1, v2 = 1;
        for (int i = 1; i <= n; i ++) {
            mul(v1, walk(a[i], l, l + k));
            mul(v2, walk(a[i], l + 1, l + k));
        }
        add(ans, v1);
        dec(ans, v2);
    }
    return ans;
}

int main() {
    std::ios::sync_with_stdio(0);
    std::cin.tie(0);

    std::cin >> n >> m >> k;

    bc.init(V * 2);
    for (int i = 0; i <= V * 2; i ++) {
        if (i) add(pre[i], pre[i - 1]);
        if (i % 2 == m % 2) {
            add(pre[i], bc.binom(m, (i + m) / 2));
        }
    }

    std::vector<int> a(n + 1), b(n + 1);
    for (int i = 1; i <= n; i ++) {
        int x, y;
        std::cin >> x >> y;

        a[i] = x + y, b[i] = x - y;

        // std::cout << a[i] << ' ' << b[i] << '\n';
    }

    // std::cout << calc(a) << ' ' << calc(b) << '\n';

    int ans = 1ll * calc(a) * calc(b) % mod;
    std::cout << ans << '\n';

    return 0;
}

/**
 *  心中无女人
 *  比赛自然神
 *  模板第一页
 *  忘掉心上人
**/
```