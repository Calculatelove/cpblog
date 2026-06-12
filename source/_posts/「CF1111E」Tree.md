---
title: 「CF1111E」Tree
date: 2025-03-16 00:00:01
updated: 2026-03-04 00:00:01
categories: Codeforces
tags:
  - 「数学」
  - 二项式反演
  - 树状数组
  - 换根
---

# Description

Link：[CF1111E](https://codeforces.com/contest/1111/problem/E)

{% note default %}

给出一棵包含 $n$ 个点的树。

有 $Q$ 次询问，每次询问给出三个整数 $k, m, r$，随后给出 $k$ 个树上的节点 $a_1, a_2, \dots, a_k$。假设树的根为 $r$，我们需要将这 $k$ 个节点划分成至多 $m$ 组，并且：
- 每个节点必须恰好属于一个组，每个组至少包含一个节点。
- 在任意组内，不能存在两个不同的节点，使得其中一个点是另一个点的祖先。

你需要求出分组的方案数，答案对 $10^9 + 7$ 取模。

数据范围：$1 \leq n, Q \leq 10^5$，$1 \leq k, r \leq n$，$1 \leq m \leq \min(300, k)$，$1 \leq \sum k \leq 10^5$。

时空限制：$1.5$s / $256$MiB。

{% endnote %}

<!-- more -->

# Solution

考虑计数。设 $c_1, c_2, \dots, c_k$ 分别表示 $a_1, a_2, \dots, a_k$ 到根节点 $r$ 的路径上有多少个关键点。则将 $k$ 个关键点分进**至多** $m$ 个有标号组的方案数为

$$
f(m) = \prod_{i = 1}^k (m - c_i)
$$

但由于组与组之间不区分（无标号），设 $g(m)$ 表示将 $k$ 个关键点分进恰好 $m$ 个有标号组的方案数，则答案应为 $\sum_{i = 1}^n \frac{g(i)}{i!}$。

由二项式反演

$$
f(i) = \sum\limits_{j = 1}^i \binom{i}{j} g(j) \iff g(i) = \sum\limits_{j = 1}^i (-1)^{i - j} \binom{i}{j} f(j)
$$

故先求出 $f(i)$，再通过二项式反演求出 $g(i)$，最后计算 $\sum_{i = 1}^n \frac{g(i)}{i!}$ 即可。

至于 $c_i$ 的处理，可以在 dfs 序上使用树状数组维护，并使用换根 Trick。

时间复杂度 $\mathcal{O}(\sum k \log n + \sum km)$。