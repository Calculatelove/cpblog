---
title: 「CF1444C」Team-Building
date: 2025-03-17 00:00:04
updated: 2026-03-05 15:03:25
categories: Codeforces
tags:
---

# Description

Link：[CF1444C](https://codeforces.com/contest/1444/problem/C)

{% note default %}

给出一张包含 $n$ 个点 $m$ 条边的无向图，每个点有一个颜色 $c_i$ ($1 \leq c_i \leq k$)，求有多少组颜色对 $a, b$ ($a < b$) 满足保留颜色为 $a$ 或 $b$ 的节点的导出子图是二分图。

数据范围：$1 \leq n \leq 5 \times 10^5$，$0 \leq m \leq 5 \times 10^5$，$2 \leq k \leq 5 \times 10^5$。

时空限制：$3$s / $500$MiB。

{% endnote %}

<!-- more -->

# Solution

本题要求的是，在 $k$ 组点中选出两组点，使得它们的导出子图是二分图的方案数。称端点属于同一个组的边为同组边，端点不属于同一个组的边为异组边。

首先先将所有的同组边连上，并做一次二分图染色，求出每个点 $u$ 所属于的连通块编号 $\mathrm{belong}_u$ 以及二分图染色的颜色 $\mathrm{col}_u$。

正难则反，考虑求出 “两组点原本均是二分图，但在两组点之间连边过后不是二分图” 的数量。这样的两组点有一个前提条件是两组点之间有边，故数量已经缩小到了 $\mathcal{O}(m)$ 级别。

考虑将类型相同的异组边（端点属于的两个组相同）放在一起考虑。考虑建一张新图，对于一个异组边 $(x, y)$：
- 若 $\mathrm{col}_x = \mathrm{col}_y$，则连边 $(\mathrm{belong}_x, \mathrm{belong}_y, 1)$，表示 $x, y$ 所在的连通块的颜色相反。
- 若 $\mathrm{col}_x \neq \mathrm{col}_y$，则连边 $(\mathrm{belong}_x, \mathrm{belong}_y, 0)$，表示 $x, y$ 所在的连通块的颜色相同。

对新图再进行一遍类二分图染色即可，若染色出现矛盾则即为所求。