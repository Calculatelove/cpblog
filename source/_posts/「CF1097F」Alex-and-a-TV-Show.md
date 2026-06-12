---
title: 「CF1097F」Alex and a TV Show
date: 2025-03-15 00:00:03
updated: 2026-03-04 00:00:03
categories: Codeforces
tags:
  - 「数学」
  - bitset
  - 莫比乌斯反演
---

# Description

Link：[CF1097F](https://codeforces.com/problemset/problem/1097/F)

{% note default %}

维护 $n$ 个初始为空的可重集。有 $Q$ 次操作，每次操作形如以下四种之一：
- `1 x v`：令集合 $x$ 等于 $\{v\}$。
- `2 x y z`：令集合 $x$ 等于集合 $y$ 与 $z$ 的并。
- `3 x y z`：令集合 $x$ 等于集合 $y$ 与 $z$ 的积，$A \times B = \{\gcd(a, b) \mid a \in A, b \in B\}$。
- `4 x v`：询问 $v$ 在集合 $x$ 中出现次数模 $2$ 的结果。

数据范围：$1 \leq n \leq 10^5$，$1 \leq q \leq 10^6$，$1 \leq v \leq 7000$。

时空限制：$3$s / $250$MiB。

{% endnote %}

<!-- more -->

# Solution

注意到 $v \leq 7000$ 很小。设 $a[x][v]$ 表示集合 $x$ 中，$v$ 的倍数的出现次数 mod 2 的值。使用 `std::bitset` 来维护 $a[x]$。

- `1 x v`：枚举 $v$ 的因数更新 $a[x]$ 即可。
- `2 x y z`：$a[x] \gets a[y] \mathbin{\mathrm{xor}} a[z]$。
- `3 x y z`：$a[x] \gets a[y] \mathbin{\mathrm{and}} a[z]$。
- `4 x v`：设 $f(v)$ 表示 $v$ 的倍数的出现次数，设 $g(v)$ 表示 $v$ 的出现次数，则
$$
f(v) = \sum\limits_{v \mid d} g(d) \iff g(v) = \sum\limits_{v \mid d} f(d) \mu\left( \frac{d}{v} \right)
$$

对每个数 $v$ 再开个 `std::bitset` $h[v]$，其中 $h[v][v \cdot d] = \mu(d) \bmod 2$。答案即为 $(a[x] \mathbin{\mathrm{and}} h[v]).\mathrm{count}() \bmod 2$。

时间复杂度 $\mathcal{O}(q(\sqrt{v} + \frac{v}{w}))$。