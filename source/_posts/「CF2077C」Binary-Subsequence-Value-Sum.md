---
title: 「CF2077C」Binary Subsequence Value Sum
date: 2025-03-10 00:00:03
updated: 2026-03-02 00:00:03
categories: Codeforces
tags:
  - 「数学」
  - 组合数学
  - 多项式
---

# Description

Link：[CF2077C](https://codeforces.com/contest/2077/problem/C)

{% note default %}

对于一个二进制字符串 $v$，定义其分数为

$$
\max_{0 \leq i \leq |v|} \{ F(v, 1, i) \times F(v, i + 1, |v|) \}
$$

其中 $F(v, l, r) = r - l + 1 - 2 \times \mathrm{zero}(v, l, r)$，这里 $\mathrm{zero}(v, l, r)$ 表示子串 $v[l : r]$ 中 `0` 的数量。

给出一个长度为 $n$ 的二进制字符串 $s$。

有 $Q$ 次操作，每次操作都会给出一个 $i$ ($1 \leq i \leq n$)，你需要将 $s_i$ 取反。每次操作结束后，你都需要求出 $s$ 的所有非空子序列的得分之和。答案对 $998244353$ 取模。

数据范围：$1 \leq n \leq 2 \times 10^5$，$1 \leq q \leq 2 \times 10^5$。

时空限制：$3$s / $256$MiB。

{% endnote %}

<!-- more -->

# Solution

将字符 `1` 看成 $+1$，将字符 `0` 看成 $-1$，则 $F(v, l, r)$ 即为区间 $[l, r]$ 的权值之和。

考虑式子
$$
\begin{aligned}
& \max\limits_{0 \leq i \leq n} \{ F(v, 1, i) \times F(v, i + 1, n) \} \\
= & \max\limits_{0 \leq i \leq n} \{ F(v, 1, i) \times (F(v, 1, n) - F(v, 1, i)) \}
\end{aligned}
$$

这是一个二次函数的形式，显然 $F(v, 1, i)$ 在 $\left\lfloor \frac{F(v, 1, n)}{2} \right\rfloor$ 处取到最值，且前缀的 $F$ 值一定构成一个连续的区间，故 $\left\lfloor \frac{F(v, 1, n)}{2} \right\rfloor$ 一定可以取到。

## 算法一

设 $F(v, 1, n) = p$，则字符串 $v$ 的贡献为 $\frac{p^2 - [p \bmod 2 = 1]}{4}$。$[p \bmod 2 = 1]$ 等价于子序列的长度为奇数，于是共有 $2^{n - 1}$ 个这样的子序列。

注意到，一个字符串的权值仅和该字符串的 `0` 与 `1` 个数相关，字符的顺序是不重要的。设 $a$ 表示全局 `1` 的个数，则 $n - a$ 表示全局 `0` 的个数。

设字符 `1` 选了 $i$ 个，字符 `0` 选了 $j$ 个。则 $p = i - j$，进一步 $p^2 = (i - j)^2 = i^2 + j^2 - 2ij$。可以分别计算这三个部分的贡献。

$i^2$ 的贡献为

$$
\begin{aligned}
& \sum_{i = 0}^{a} i^2 \binom{a}{i}2^{n - a} \\
= & a(a + 1)2^{a - 2}2^{n - a} \\
= & a(a + 1)2^{n - 2}
\end{aligned}
$$

同理，$j^2$ 的贡献为 $(n - a)(n - a + 1)2^{n - 2}$。

$2ij$ 的贡献为

$$
\begin{aligned}
& \sum_{i = 0}^{a} \sum_{j = 0}^{n - a} \binom{a}{i}\binom{n - a}{j}2ij \\
& = 2\left( \sum_{i = 0}^a \binom{a}{i}i \right)\left( \sum_{j = 0}^{n - a}\binom{n - a}{j}j \right) \\
& = a(n - a)2^{n - 1}
\end{aligned}
$$

于是答案为

$$
\frac{a(a + 1)2^{n - 2} + (n - a)(n - a + 1)2^{n - 2} - a(n - a)2^{n - 1} - 2^{n - 1}}{4}
$$

## 算法二

记 $\mathrm{value}(x) = \lfloor \frac{x}{2} \rfloor (x - \lfloor \frac{x}{2} \rfloor)$，则字符串 $v$ 的贡献为 $\mathrm{value}(F(v, 1, n))$。

注意到，一个字符串的权值仅和该字符串的 `0` 与 `1` 个数相关，字符的顺序是不重要的。

设 $a$ 表示全局 `1` 的个数，则 $n - a$ 表示全局 `0` 的个数。故子序列的权值 $x$ 的取值范围为 $[a - n, a]$。

对于一个权值 $x$，考虑计算有多少个子序列的权值等于 $x$，枚举子序列 `1` 的个数为 $i$，则 `0` 的个数为 $i - x$。则答案为

$$
\begin{aligned}
& \sum_{x = a - n}^a \mathrm{value}(x) \cdot \left( \sum_i \binom{a}{i}\binom{n - a}{i - x} \right) \\
= & \sum_{x = a - n}^a \mathrm{value}(x) \cdot \left( \sum_i \binom{a}{a - i}\binom{n - a}{i - x} \right) \\
= & \sum_{x = a - n}^a \mathrm{value}(x) \cdot \binom{n}{a - x}
\end{aligned}
$$

发现这是一个和 $a$ 有关的卷积式，使用 NTT 即可。~~给 998244353 磕一个。~~