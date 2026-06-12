---
title: 「CF594D」REQ
date: 2025-03-12 00:00:02
updated: 2026-03-04 00:00:02
categories: Codeforces
tags:
  - 「数据结构」
  - 树状数组
---

# Description

Link：[CF594D](https://codeforces.com/problemset/problem/594/D)

{% note default %}

给出一个长度为 $n$ 的数组 $a$。

有 $Q$ 次查询，每次查询给出两个正整数 $l, r$ ($1 \leq l \leq r \leq n$)，你需要求出

$$
\varphi \left( \prod_{i = l}^{r} a_i \right)
$$

答案对 $10^9 + 7$ 取模。

数据范围：$1 \leq n, Q \leq 2 \times 10^5$，$1 \leq a_i \leq 10^6$，$1 \leq l \leq r \leq n$。

时空限制：$3$s / $256$MiB。

{% endnote %}

<!-- more -->

# Solution

对于一个质数 $p$，若 $p$ 是区间 $[l, r]$ 中某个数的因数，则欧拉函数还要再乘上 $\frac{p - 1}{p}$。而 $10^6$ 以内，一个数最多只有 $7$ 个质因数。于是我们要数出区间 $[l, r]$ 中作为因数出现的质数 $p$ 所带来的影响。

一开始写了一个简单的莫队，可惜 TLE 了。

考虑扫描线，将所有的询问 $l, r$ 按照右端点 $r$ 从小到大排序。对于一个质数 $p$，设其在区间 $[1, r]$ 中最后一个作为因数出现的位置为 $x$，则只要左端点 $l \leq x$，答案就需要乘上 $\frac{p - 1}{p}$。

于是我们**将 $\frac{p - 1}{p}$ 的贡献挂在位置 $x$ 处**，询问区间 $[l, r]$ 只需求出区间 $[l, r]$ 的贡献之积即可。使用树状数组维护（树状数组也是可以求前缀积的）。