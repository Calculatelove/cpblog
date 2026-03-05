---
title: 「CF917D」Stranger Trees
date: 2025-03-24 00:00:01
updated: 2026-03-05 15:49:00
categories: Codeforces
tags: 
  - DP
  - Prufer 序列
---

# Description

Link：[CF917D](https://codeforces.com/problemset/problem/917/D)

{% note default %}

给出一棵大小为 $n$ 的树。

对于每个 $k$ ($0 \leq k \leq n - 1$)，你需要求出有多少个大小为 $n$ 的带标号树与原树有恰好 $k$ 条重合的边。答案对 $10^9 + 7$ 取模。

数据范围：$1 \leq n \leq 100$。

时空限制：$1$s / $250$MiB。

{% endnote %}

<!-- more -->

# Solution

<span class="highlight">扩展 Cayley 公式</span>：对于一个包含 $n$ 个点 $m$ 条边的森林，设其有 $k$ 个连通块，大小分别为 $s_1, s_2, \cdots, s_k$，则可以使得森林变成树的加边方式恰有

$$
n^{k - 2}\prod_{i = 1}^k s_i
$$

考虑**钦定**原树中的 $i$ 条边（则会产生 $n - i$ 个连通块）已经被选进生成树时的方案数为 $\mathrm{cnt}(i)$，**恰好**原树的 $i$ 条边被选进生成树的方案数为 $g(i)$。则有

$$
\mathrm{cnt}(i) = \sum\limits_{j = i}^{n - 1} \binom{j}{i} g(j) \iff g(i) = \sum\limits_{j = i}^{n - 1} (-1)^{j - i} \binom{j}{i} \mathrm{cnt}(j)
$$

于是可以考虑先求出 $\mathrm{cnt}$ 再反演出 $g$。

注意到 $\prod_{i = 1}^k a_i$ 不太好直接处理。Trick：**每个部分大小的乘积**等于**每个部分各选出一个元素的方案数**。

考虑 dp，设 $f(u, i, 0/1)$ 表示考虑到了 $u$ 的子树，已经选出了 $i$ 个连通块，且 $u$ 所在的连通块 暂未 / 已经 选出一个点的方案数。枚举 $u$ 的每一个儿子 $v$，有转移
- 不合并连通块
$$
f'(u, i + j, 0) \gets_{+} f(u, i, 0) \cdot f(v, j, 1) \\
f'(u, i + j, 1) \gets_{+} f(u, i, 1) \cdot f(v, j, 1)
$$
- 合并连通块
$$
f'(u, i + j - 1, 0) \gets_{+} f(u, i, 0) \cdot f(v, j, 0) \\
f'(u, i + j - 1, 1) \gets_{+} f(u, i, 1) \cdot f(v, j, 0) \\
f'(u, i + j - 1, 1) \gets_{+} f(u, i, 0) \cdot f(v, j, 1)
$$

于是 $\mathrm{cnt}(i) = f(1, n - i, 1) \cdot n^{n - i - 2}$。

时间复杂度 $\mathcal{O}(n^2)$。