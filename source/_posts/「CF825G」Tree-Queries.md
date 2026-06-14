---
title: 「CF825G」Tree Queries
date: 2025-03-13 00:00:01
updated: 2026-03-04 00:00:01
categories: Codeforces
tags:
  - 树上问题
---

# Description

Link：[CF825G](https://codeforces.com/problemset/problem/825/G)

{% note default %}

给出一棵包含 $n$ 个点的树，初始时所有点均为白色。

有 $Q$ 次操作，每次操作形如一下的两种：
- `1 x`：将点 $x$ 改成黑色。
- `2 x`：对于点 $x$，找出编号最小的点 $y$，使得 $y$ 位于 $x$ 到某个黑色顶点的简单路径上。

**本题强制在线**。保证第一次操作的类型为操作 1。

数据范围：$3 \leq n, Q \leq 10^6$。

时空限制：$3$s / $256$MiB。

{% endnote %}

<!-- more -->

# Solution

此题相当于维护黑点点集 $S$ 的**最小连通子图**的**节点编号最小值**。

在此题中，黑点不会再重新变回白点。考虑以第一次修改的黑点为根节点 $\mathrm{root}$，通过一次 dfs 求出根节点到每个节点 $x$ 的路径上，节点编号的最小值 $\mathrm{dat}[x]$。

每次新加入一个黑点 $x$，就将 $x$ 到 $\mathrm{root}$ 路径上的所有点全都加入最小连通子图中（即用 $\mathrm{dat}[x]$ 更新答案），显然任意两个黑点之间的所有点均被加入最小连通子图，故该种处理方式是正确的。且由于最小值具有可重复贡献性，故不用考虑去重。