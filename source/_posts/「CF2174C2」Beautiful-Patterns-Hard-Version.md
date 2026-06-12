---
title: 「CF2174C2」Beautiful Patterns (Hard Version)
date: 2025-12-08 17:36:14
updated: 2025-12-08 17:36:14
categories: Codeforces
tags:
  - 「数学」
  - 回文串
  - 期望
---

# Description

Link：[CF2174C2](https://codeforces.com/contest/2174/problem/C2)

{% note default %}

已知一个字符串 $S$ 的长度为 $n$，字符集大小为 $m$。现在均匀随机地生成该字符串 $S$。

记随机变量 $X$ 表示该字符串 $S$ 中回文子串的数量，求 $E(X^2)$。答案对给定质数 $p$ 取模。

数据范围：$1 \leq n \leq 2 \times 10^5$，$1 \leq m \leq 10^7$，$m < p < 10^9$。

时空限制：$2$s / $512$MiB。

{% endnote %}

<!-- more -->

# Solution

平方的期望是一个经典 Trick。记 $x_{l, r}$ 表示区间 $[l, r]$ 是否回文，则

$$
\begin{aligned}
E(X^2) & = E\left(\left( \sum_{1 \leq l \leq r \leq n} x_{l, r} \right)^2 \right) \\
& = E\left( \sum_{l_1 \leq r_1, l_2 \leq r_2} x_{l_1, r_1}\cdot x_{l_2, r_2} \right) \\
& = \sum_{l_1 \leq r_1, l_2 \leq r_2} E\left( x_{l_1, r_1}\cdot x_{l_2, r_2} \right)
\end{aligned}
$$

相当于是计算，任意取出两个区间，均为回文串的概率之和。

若两个区间共用一个回文中心，则概率为较长串为回文串的概率。

若两个区间不共用一个回文中心，有一个很深刻的事实：**两个区间是否回文是相互独立的**。

{% note info %}

<details>
<summary> 简单证明 </summary>

对于一个回文区间 $[l, r]$，相当于是将二元组 $(l, r), (l + 1, r - 1), \dots$ 位置上的字符进行合并。显然每次有效的合并（两点在之前不连通），都会导致自由元个数减少一个。

我们可以证明：**每次合并都有效，即这样连边一定不会出现环**。所以自由元的损失个数即为实际边数。

对于一个回文中心来说，产生的边的端点之和必定为 $l + r$。记两个回文中心的端点之和分别为 $C_1, C_2$。则 $x$ 应用变换 1 得到 $C_1 - x$，应用变换 2 得到 $C_2 - x$。

假设连边会出现环，则一定存在一个 $x$ 经过一条**简单路径**再次回到 $x$。由于同一种变换不能连续使用（不然就形成了重边），不妨设变换 1, 2 交替使用：
- 偶数次变换：得到 $x + k(C_2 - C_1)$，由于 $C_1 \neq C_2$，因此**必定不等于 $x$**。
- 奇数次变换：假设回到了 $x$，则必然是从 $C_1 - x$ 回到 $x$ 的。**但这是一条重边，而不是一个环**。

Q.E.D

</details>

{% endnote %}

然后按照是否共用一个回文中心，分两类计算贡献即可。

时间复杂度 $\mathcal{O}(n)$。

```c++
#include <bits/stdc++.h>

using i64 = long long;
using u64 = unsigned long long;

#define debug(a) std::cout << #a << "=" << (a) << ' '

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

const int N = 200100;

int n, m, mod;

inline void add(int &x, const int &y) {
    x += y;
    if (x >= mod) {
        x -= mod;
    }
}
inline void dec(int &x, const int &y) {
    x -= y;
    if (x < 0) {
        x += mod;
    }
}
inline int norm(int x) {
    x %= mod;
    return x < 0 ? x + mod : x;
}

constexpr int qpow(int a, int b, int p) {
    int ans = 1;
    for (; b; b >>= 1) {
        if (b & 1) ans = 1ll * ans * a % p;
        a = 1ll * a * a % p;
    }
    return ans;
}

int im;

int power[N], pre[N];
int s0[N], s1[N];

void work() {
    std::cin >> n >> m >> mod;

    im = qpow(m, mod - 2, mod);

    power[0] = 1;
    for (int i = 1; i <= n; i ++) {
        power[i] = 1ll * power[i - 1] * im % mod;
    }

    pre[0] = 1;
    for (int i = 1; i <= n; i ++) {
        add(pre[i] = pre[i - 1], power[i]);
    }

    s0[0] = 0;
    for (int i = 1; i <= n; i ++) {
        add(s0[i] = s0[i - 1], 1ll * power[i] * i % mod);
    }

    s1[0] = 0;
    for (int i = 1; i <= n; i ++) {
        add(s1[i] = s1[i - 1], 1ll * power[i] * (i - 1) % mod);
    }

    int ans1 = 0, ans2 = 0, sum = 0;
    for (int i = 1; i <= n; i ++) { // 0~u
        int u = std::min(i - 1, n - i);
        int cur = pre[u];

        add(ans1, cur);
        add(ans2, s0[u]);

        add(ans2, 1ll * cur * sum % mod);
        add(sum, cur);
    }
    for (int i = 1; i < n; i ++) { // 1~u
        int u = std::min(i, n - i);
        int cur = norm(pre[u] - 1);

        add(ans1, cur);
        add(ans2, s1[u]);

        add(ans2, 1ll * cur * sum % mod);
        add(sum, cur);
    }

    int ans = (ans1 + 2ll * ans2) % mod;
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