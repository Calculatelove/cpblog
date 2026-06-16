---
title: 「CF2159C」Twin Polynomials
date: 2025-10-13 14:30:34
updated: 2025-10-13 14:30:34
categories: Codeforces
tags:
  - DP
  - 结论题
---

# Description

Link：[CF2159C](https://codeforces.com/contest/2159/problem/C)

{% note default %}

对于一个多项式 $f(x) = a_0 + a_1x + a_2x^2 + \dots + a_nx^n$，当 $a_i$ ($0 \leq i \leq n$) 均为非负整数且 $a_n \neq 0$ 时，多项式 $f(x)$ 被称为 $n$ 阶有效多项式。

对于一个 $n$ 阶有效多项式 $f(x) = a_0 + a_1x + a_2x^2 + \dots + a_nx^n$，其孪生多项式 $g(x)$ 定义为

$$
g(x) = \sum_{i = 0}^n i \cdot x^{a_i}
$$

当且仅当 $f(x) = g(x)$ 时，$f(x)$ 被称为酷多项式。

给出一个不完整的 $n$ 阶有效多项式 $f(x) = a_0 + a_1x + a_2x^2 + \dots + a_nx^n$，$a_i$ 的一部分已经确定，剩余部分暂未确定。**保证 $a_0$ 与 $a_n$ 未确定**。

请求出确定了未确定的部分以后，能得到多少个 $n$ 阶有效酷多项式。答案对 $10^9 + 7$ 取模。

数据范围：$1 \leq n \leq 4 \times 10^5$。

时空限制：$2$s / $256$MiB。

{% endnote %}

<!-- more -->

# Solution

将条件转化一下，可得 $a_i = \sum_{a_j = i} j$。

由于 $a_0$ 对其他 $a_i$ 没有贡献，并且 $a_0$ 必定未确定，可以考虑使用 $a_0$ 来调配那些 $a_i = 0$ 的数。

{% note info %}

$f(x)$ 为酷多项式，则 $a_1, \dots, a_n$ 必定满足这些情况：
- 指向零：$a_i = 0$。
- 不动点：$a_i = i$。
- 对换：$a_i = j$ 且 $a_j = i$，其中 $i \neq j$。

其中 $a_0$ 即为所有 $a_i = 0$ 的 $i$ 之和。

证明：建立图论模型 $i \to a_i$。则对于 $a_i \neq 0$ 的部分，每个点至少有一条入边，而每个点仅有一条出边，**故 $a_i \neq 0$ 的部分必定是每个点恰好一个出边一个入边**。即为不动点与对换。

对于 $a_i = 0$ 的部分，显然也是符合要求的。

{% endnote %}

于是先根据该规则，对于已经出现的 $a_i$，尝试将 $(i, a_i)$ 配成对换。如果出现矛盾则答案为 $0$。

剩下还有一些未确定的点，考虑 dp 统计方案数。设 $f_i$ 表示有多少个 $i$ 阶有效酷多项式。考虑 $a_i$ 的三种情况，有转移

$$
f_i = 2f_{i - 1} + (i - 1)f_{i - 2}
$$

注意其中 $a_n \neq 0$，因为必须要保证这是一个 $n$ 阶有效多项式。

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

const int mod = 1e9 + 7; // 模数需根据实际问题调整

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

const int N = 400100;

int n;
int a[N];

int f[N];

void work() {
    std::cin >> n;

    for (int i = 0; i <= n; i ++) {
        std::cin >> a[i];
    }
    for (int i = 1; i <= n; i ++) {
        if (a[i] > n) {
            std::cout << 0 << '\n';
            return;
        }
        if (a[i] == i || a[i] == 0 || a[i] == -1) {
            continue;
        }
        if (a[a[i]] == -1) {
            a[a[i]] = i;
        } else if (a[a[i]] != i) {
            std::cout << 0 << '\n';
            return;
        }
    }

    int cnt = 0;
    for (int i = 1; i <= n; i ++) {
        if (a[i] == -1) {
            cnt ++;
        }
    }

    f[0] = 1, f[1] = 2;
    for (int i = 2; i <= n; i ++) {
        f[i] = (2LL * f[i - 1] + 1LL * (i - 1) * f[i - 2]) % mod;
    }

    int ans = 0;
    if (a[n] == -1) {
        ans = cnt ? (f[cnt - 1] + 1LL * (cnt - 1) * (cnt >= 2 ? f[cnt - 2] : 0)) % mod : 1;
    } else {
        ans = f[cnt];
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