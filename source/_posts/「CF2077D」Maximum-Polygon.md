---
title: 「CF2077D」Maximum Polygon
date: 2026-03-17 00:00:03
updated: 2026-03-05 14:38:41
categories: Codeforces
tags:
  - 贪心
---

# Description

Link：[CF2077D](https://codeforces.com/contest/2077/problem/D)

{% note default %}

给出一个长度为 $n$ 的数组 $a$，你需要找出**字典序最大**的子序列 $s$，使得 $s$ 可以作为多边形的边长。

当且仅当 $|s| \geq 3$ 且满足以下条件时，$s$ 可以作为多边形的边长

$$
2 \cdot \max(s_1, \cdots, s_{|s|}) < s_1 + \cdots + s_{|s|}
$$

数据范围：$3 \leq n \leq 2 \times 10^5$，$1 \leq a_i \leq 10^9$。

时空限制：$3$s / $250$MiB。

{% endnote %}

<!-- more -->

# Solution

考虑当子序列最大值 $\mathrm{mx}$ 已经确定时，要如何求出字典序最大的子序列 $s$？此时相当于要选出总和 $> 2\cdot \mathrm{mx}$ 的字典序最大的子序列。

考虑以下贪心过程：从左往右扫描每一个 $a_i$，设 $\mathrm{sum}$ 表示当前 $s$ 元素之和。
- 取出当前 $s$ 的最后一个元素 $x$，若 $a_i > x$ 且 $\mathrm{sum} - x + \sum_{j = i}^n a_j > 2 \cdot \mathrm{mx}$，则可以将 $x$ 替换成 $a_i$。重复该过程直到无法替换或 $s$ 为空。
- 将 $a_i$ 放入当前 $s$ 的末尾。

可以证明，子序列 $s$ 的最大值一定是 $\{a_i\}$ 的前 $\log_2 A$ 大其中之一。

考虑一个序列 $v$，其中 $v$ 的**任何子序列**都不可以作为多边形的边长。则升序排序过后一定满足 $v_i \geq \sum_{j = 1}^{i - 1}v_j$，为了最大化 $v$ 的长度，有 $v_1 = 1$ 且 $v_i = 2^{i - 2}$（$i \geq 2$）。

故满足任何子序列都不可以作为多边形边长的序列 $v$，长度不超过 $\log_2 A$。

换言之，**长度超过 $\log_2 A$ 的序列 $v$，必定有一个子序列可以作为多边形的边长**。

假设答案的子序列 $s$ 的最大值不是 $\{a_i\}$ 的前 $\log_2 A$ 大其中之一，则将前 $\log_2 A$ 大构成的子序列 $t$ 取出，由上述推导，$t$ 必定有一个子序列 $t'$ 可以作为多边形的边长。将 $t'$ 插入子序列 $s$ 中，显然满足条件的同时字典序更大。

时间复杂度 $\mathcal{O}(n \log A)$。 