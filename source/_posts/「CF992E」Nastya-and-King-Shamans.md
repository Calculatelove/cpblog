---
title: 「CF992E」Nastya and King-Shamans
date: 2025-03-15 00:00:02
updated: 2026-03-04 00:00:02
categories: Codeforces
tags:
  - 线段树
---

# Description

Link：[CF992E](https://codeforces.com/problemset/problem/992/E)

{% note default %}

给出一个长度为 $n$ 的数组 $a$，记 $s_i = \sum_{j = 1}^i a_j$。

有 $Q$ 次操作，每次操作给出两个整数 $p, x$，表示将 $a_p$ 赋成 $x$。每次操作后，你都需要判断是否存在一个位置 $i$ 满足 $a_i = s_{i - 1}$。若存在，输出任意一个满足条件的 $i$。

数据范围：$1 \leq n, Q \leq 2\times 10^5$，$0 \leq a_i \leq 10^9$，$1 \leq p \leq n$，$0 \leq x \leq 10^9$。

时空限制：$3$s / $256$MiB。

{% endnote %}

<!-- more -->

# Solution

注意到 $a_i \geq 0$。若 $a_i = s_{i - 1}$，则 $s_i = 2s_{i - 1}$。故满足要求的位置个数是 $\mathcal{O}(\log a)$ 级别的。

进一步，考虑 $a_i \geq s_{i - 1}$ 即 $s_i \geq 2s_{i - 1}$。满足该要求的位置个数也是 $\mathcal{O}(\log a)$ 级别的。

使用线段树维护 $a_i - s_{i - 1}$ 的最大值。每次暴力地遍历整棵线段树，遇到最大值 $< 0$ 的区间就返回，遇到一个叶子就判断该点对应的 $a_i - s_{i - 1}$ 是否等于 $0$。

时间复杂度 $\mathcal{O}(n \log n \log a)$。