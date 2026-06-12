---
title: 「CF1174E」Ehab and the Expected GCD Problem
date: 2025-03-15 00:00:01
updated: 2026-03-04 00:00:01
categories: Codeforces
tags:
  - DP
---

# Description

Link：[CF1174E](https://codeforces.com/problemset/problem/1174/E)

{% note default %}

对于一个长度为 $n$ 的排列 $p$，定义 $f(p)$ 表示：令 $g_i$ 表示 $p_1, \dots, p_i$ 的最大公约数，则 $f(p)$ 表示 $g_1, g_2, \dots, g_n$ 中不同元素的个数。

令 $f_{\max}(n)$ 表示所有关于 $n$ 的排列 $p$ 中的 $f(p)$ 最大值，求有多少个关于 $n$ 的排列 $p$ 满足 $f(p) = f_{\max}(n)$。答案对 $10^9 + 7$ 取模。

数据范围：$2 \leq n \leq 10^6$。

时空限制：$2$s / $256$MiB。

{% endnote %}

<!-- more -->

# Solution

本质不同的前缀 gcd 是 $\mathcal{O}(\log n)$ 级别的，因为当前缀 gcd 改变时，新的 gcd 一定为旧的 gcd 的约数。

考虑在唯一分解角度下观察 gcd，设 gcd 为 $\sum p_i^{c_i}$，为了使得本质不同的前缀 gcd 最多，当 gcd 变化时，肯定是选择某一个 $c_i \gets c_i - 1$。故为了使得本质不同的 gcd 最多，就是要使得排列的第一个数的 $\sum c_i$ 最多。

首先，肯定不会存在质因子 $p_i \geq 5$，因为此时可以将 $p_i$ 换成 $2^2$，$\sum c_i$ 更大且仍然合法。

其次，肯定不会存在 $3$ 的次数 $\geq 2$，因为此时可以将 $3^2$ 换成 $2^3$，$\sum c_i$ 更大且仍然合法。

故第一个数只能被表示成 $2^x3^y$，其中 $y \in \{0, 1\}$。状态数为 $\mathcal{O}(\log n)$。

设 $f(i, x, y)$ 表示，填到排列的前 $i$ 位，且当前的 gcd 为 $2^x3^y$ 时的方案数。设 $\mathrm{calc}(x) = \left\lfloor \frac{n}{x} \right\rfloor$ 表示 $1, \dots, n$ 中 $x$ 的倍数的个数。

$$
f(i + 1, x, y) \gets_{+} f(i, x, y) \cdot (\mathrm{calc}(2^x3^y) - i) \\
f(i + 1, x - 1, y) \gets_{+} f(i, x, y) \cdot (\mathrm{calc}(2^{x - 1}3^y) - \mathrm{calc}(2^x3^y)) \\
f(i + 1, x, y - 1) \gets_{+} f(i, x, y) \cdot (\mathrm{calc}(2^x3^{y - 1}) - \mathrm{calc}(2^x3^y))
$$

初态：记 $u = \lfloor \log_2 n \rfloor$，则有 $f(1, u, 0) = 1$；若 $2^{u - 1}3 \leq n$ 时，还有 $f(1, u - 1, 1) = 1$。

终态：$f(n, 0, 0)$。