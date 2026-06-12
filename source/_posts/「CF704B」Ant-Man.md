---
title: 「CF704B」Ant Man
date: 2025-03-12 00:00:03
updated: 2026-03-04 00:00:03
categories: Codeforces
tags:
  - DP
---

# Description

Link：[CF704B](https://codeforces.com/problemset/problem/704/B)

{% note default %}

有 $n$ 个元素，第 $i$ 个元素有五个参数 $x_i, a_i, b_i, c_i, d_i$。

你需要求出一个 $1 \sim n$ 的排列 $p$，满足 $p_1 = s, p_n = e$，同时最小化这个排列的权值。一个排列的权值定义为 $\sum_{i = 1}^{n - 1} f(p_i, p_{i + 1})$，其中
- 若 $i > j$，则 $f(i, j) = x_i - x_j + c_i + b_j$。
- 若 $i < j$，则 $f(i, j) = x_j - x_i + d_i + a_j$。

你只需要求出排列的最小权值即可。

数据范围：$1 \leq n \leq 5 \times 10^3$，$s \neq e$，$1 \leq x_1 < x_2 < \dots < x_n \leq 10^9$，$1 \leq a_i, b_i, c_i, d_i \leq 10^9$。

时空限制：$4$s / $250$MiB。

{% endnote %}

<!-- more -->

# Solution

连续段 dp 的一个好题。

注意到，相邻的两个数 $i, j$ 的贡献是独立的，只取决于他们之间的大小关系。这有助于**代价提前计算**。

**考虑从小到大加数**。不可当成仅一个连续段处理的原因是，插入一个数会破坏原有的相邻大小关系，无法计算代价。于是考虑连续段 dp，设 $f(i, j)$ 表示填了数字 $1 \sim i$，形成了 $j$ 个连续段时的最小代价。有四种转移：
- 将 $i$ 独自新开一个连续段：此时 $i$ 比两边都小。
- 将 $i$ 接在某个连续段的左边：此时 $i$ 比左小，比右大。
- 将 $i$ 接在某个连续段的右边：此时 $i$ 比左大，比右小。
- 通过 $i$ 将某两个连续段合并：此时 $i$ 比两边都大。

特别地，在 $p_1 = s, p_n = e$ 的影响下，某些情况下的某些转移是不合法的。