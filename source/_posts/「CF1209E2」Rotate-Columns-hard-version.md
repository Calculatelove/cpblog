---
title: 「CF1209E2」Rotate Columns (hard version)
date: 2025-03-10 00:00:02
updated: 2026-03-02 00:00:02
categories: Codeforces
tags:
  - 「DP」
  - 状压 DP
---

# Description

Link：[CF1209E2](https://codeforces.com/problemset/problem/1209/E2)

{% note default %}

给出一个 $n \times m$ 的矩阵 $a$。

你可以进行若干次操作。每次操作，你可以选择任意一列，并循环移位该列中的元素。

设 $r_i$ 表示第 $i$ 行的最大值，求 $\sum_{i = 1}^n r_i$ 的最大值。

数据范围：$1 \leq n \leq 12$，$1 \leq m \leq 2000$，$1 \leq a_{i, j} \leq 10^5$。

时空限制：$3$s / $512$MiB。 

{% endnote %}

<!-- more -->

# Solution

真正有效的列，一定在列最大值前 $n$ 大的那些列之中。容易使用反证法证明。

考虑一列一列处理，在每一列中钦定有哪些位置为对应行的最大值。由于此题要求的是最大值之和的最大值，故在统计答案的过程中，我们**不必保证每次钦定的最大值一定是真实的最大值。因为不成立的一定不优**。

设 $f(j, S)$ 表示考虑到了前 $j$ 列，其中每行的最大值确定状态为一个二进制数 $S$（若 $S$ 的第 $i$ 位为 $1$，则表示第 $i$ 行的最大值已经确定），则有

$$
f(j, S) = \max\limits_{T \subseteq S} \{ f(j - 1, T) + \mathrm{value}(j, S \setminus T) \}
$$

其中 $\mathrm{value}(j, S)$ 表示第 $j$ 列中，所有行号属于 $S$ 的位置，在 $0 \sim n - 1$ 次循环移位下元素之和的最大值。

时间复杂度 $\mathcal{O}(n3^n + n^22^n)$。