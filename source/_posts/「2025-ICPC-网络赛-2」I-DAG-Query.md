---
title: 「2025 ICPC 网络赛 2」I. DAG Query
date: 2025-09-14 23:07:48
updated: 2025-09-14 23:07:48
categories: ICPC
tags:
  - 拉格朗日插值
---

# Description

Link：[QOJ 14322](https://qoj.ac/contest/2524/problem/14322)

{% note default %}

**This is an interactive problem.**

给出一个 $n$ 个点 $m$ 条边的有向无环图（DAG）。你会得到该 DAG 的结构，但不会知道 DAG 中每条边的边权。

在任意一对顶点 $s$ 与 $t$ 之间可能存在多条路径，我们将一条路径的权值定义为该路径上所有边权的乘积。记 $f(s, t, c)$ 表示从 $s$ 到 $t$ 的所有不同路径，在所有边权都乘以 $c$ 情况下的权值之和，对 $998244353$ 取模。

你可以进行至多 $999$ 次询问，每次询问你需要给出参数 $s, t, c$，交互器会返回 $f(s, t, c)$ 的值。

最后，交互器会给出一个参数 $k$，你需要确定 $f(1, n, k)$ 的值。

数据范围：$1 \leq n \leq 10^3$，$1 \leq m \leq 5 \times 10^3$。

时空限制：$3$s / $1024$MiB。

{% endnote %}

<!-- more -->

没啥意思的一道题，记录一下是为了扫清拉格朗日插值中的一些误区。

# Solution

设 $a_i$ 表示从 $1$ 到 $n$ 的所有长度为 $i$ 的路径的权值之和。此时会发现

$$
f(1, n, x) = \sum\limits_{i = 0}^{n - 1}a_ix^i
$$

容易看出 $f(1, n, x)$ 是一个关于 $x$ 的 $n - 1$ 次多项式，需要 $n$ 个点值才能拉插。

使用 $n - 1$ 次询问，得到 $x = 1 \cdots n - 1$ 时 $f(1, n, x)$ 的值。再加上 $f(1, n, 0) = 0$ 这一个点值，刚好凑齐 $n$ 个点值，直接拉插！

{% note warning %}

误区 1：$f(1, n, x)$ 可能并不是 $n - 1$ 次多项式，显然 $1$ 到 $n$ 并不一定存在长度为 $n - 1$ 的路径，此时 $a_{n - 1} = 0$ 怎么办？

<details>
<summary> 解释 1 </summary>

确实！！！实际上给定 $n$ 个横坐标两两不同的点值，拉格朗日插值得出的多项式是**至多 $n - 1$ 次，而不是恰好 $n - 1$ 次**。

当数据恰好落在更低次的多项式上时，实际次数会更小。例如：$(1, 1), (2, 4), (3, 9), (4, 16)$，拉格朗日插值得出的多项式为 $x^2$。

但我们可以证明**拉格朗日插值 $f(x) = \sum_{i = 1}^{n} y_i \prod_{j \neq i} \frac{x - x_j}{x_i - x_j}$，是唯一的插值多项式**。

证明 1：假设 $g(x)$ 也是次数至多 $n - 1$ 次，且满足 $g(x_i) = y_i$ 的多项式。**显然 $f(x) - g(x)$ 至少有 $n$ 个不相同的零点，但一个次数至多为 $n - 1$ 的非常数多项式的零点个数不可能超过 $n - 1$ 个**，故 $f(x) - g(x)$ 恒等于 $0$。

证明 2：将 $f(x_i) = y_i$ 写成线性方程组

$$
\begin{pmatrix}
1 & x_1 & x_1^2 & \cdots & x_1^{n - 1} \\
1 & x_2 & x_2^2 & \cdots & x_2^{n - 1} \\
\vdots & \vdots & \vdots & \ddots & \vdots \\
1 & x_n & x_n^2 & \cdots & x_n^{n - 1} \\
\end{pmatrix}
\begin{pmatrix}
a_0 \\
a_1 \\
\vdots \\
a_{n - 1}
\end{pmatrix}
=
\begin{pmatrix}
y_1 \\
y_2 \\
\vdots \\
y_{n}
\end{pmatrix}
$$

注意到系数矩阵是一个范德蒙德矩阵 $M$，其行列式为

$$
\det M = \prod_{1 \leq i < j \leq n} (x_j - x_i)
$$

由于 $x_i$ 两两不同，故 $\det M \neq 0$，因此该方程组有唯一解。

</details>

{% endnote %}

{% note warning %}

误区 2：$f(1, n, 0) = 0$ 不是非常显然的事情吗？这个点值真的有效吗？

<details>
<summary> 解释 2 </summary>

当然有效！是你多虑了。只要保证 $n$ 个点值的横坐标两两不同即可。

</details>

{% endnote %}

时间复杂度 $\mathcal{O}(n^2)$。

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

void mul(int &x, int y) {
    x = 1ll * x * y % mod;
}

int qpow(int a, int b, int p) {
    int ans = 1;
    for (; b; b >>= 1) {
        if (b & 1) ans = 1ll * ans * a % p;
        a = 1ll * a * a % p;
    }
    return ans;
}

const int N = 1010;

int n, m;

int ask(int x, int y, int c) {
    std::cout << "? " << x << ' ' << y << ' ' << c << std::endl;

    int res;
    std::cin >> res;

    return res;
}

int lagrange(std::vector< std::pair<int, int> > seq, int k) {
    int n = seq.size();
    int ans = 0;
    for (int i = 0; i < n; i ++) {
        int p = 1, q = 1;
        for (int j = 0; j < n; j ++) {
            if (i == j) continue;
            mul(p, k - seq[j].first);
            mul(q, seq[i].first - seq[j].first);
        }
        ans = (ans + 1ll * p * qpow(q, mod - 2, mod) % mod * seq[i].second) % mod;
    }
    return (ans + mod) % mod;
}

int main() {
    std::ios::sync_with_stdio(0);
    std::cin.tie(0);

    std::cin >> n >> m;

    for (int i = 1; i <= m; i ++) {
        int x, y;
        std::cin >> x >> y;
    }

    std::vector< std::pair<int, int> > seq(n);
    seq[0] = {0, 0};
    for (int i = 1; i < n; i ++) {
        seq[i] = {i, ask(1, n, i)};
    }

    std::cout << "!" << std::endl;

    int c;
    std::cin >> c;

    std::cout << lagrange(seq, c) << std::endl;

    return 0;
}
```