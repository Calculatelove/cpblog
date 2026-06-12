---
title: 「CF1830D」Mex Tree
date: 2025-03-20 00:00:01
updated: 2026-03-05 15:34:37
categories: Codeforces
tags:
  - 「DP」
  - 树上背包
---

# Description

Link：[CF1830D](https://codeforces.com/contest/1830/problem/D)

{% note default %}

给出一棵包含 $n$ 个节点的树。你需要给每个节点染上数字 $0$ 或 $1$。

路径 $(u, v)$ 的权值，等于从 $u$ 到 $v$ 的最短路径上所有节点数字的 MEX。一种染色方案的权值等于所有 $1 \leq u \leq v \leq n$ 的路径 $(u, v)$ 的权值之和。

请求出所有的染色方案中，可能的最大权值。

数据范围：$1 \leq n \leq 2 \times 10^5$。

时空限制：$3$s / $256$MiB。

{% endnote %}

<!-- more -->

# Solution

考虑以同色连通块的视角处理此题。注意到横跨至少一个全 $0$ 连通块与全 $1$ 连通块的路径 $\mathrm{mex} = 2$，全 $0$ 连通块内的路径 $\mathrm{mex} = 1$，全 $1$ 连通块内的路径 $\mathrm{mex} = 0$。

于是我们反过来考虑。设一开始所有路径均有 $2$ 的贡献，对于一个大小为 $k$ 的全 $0$ 连通块有 $\frac{k(k + 1)}{2}$ 的负贡献，对于一个大小为 $k$ 的全 $1$ 连通块有 $k(k + 1)$ 的负贡献。

一个不成熟的想法是对这棵树进行黑白染色，黑白染色的负贡献 $\leq 2n$。但有时黑白染色并不是最优的，例如将两个菊花图的中心以一条边相连，此时可以将两个中心染成 $1$，将叶子染成 $0$。但我们可以通过黑白染色可以进一步约束连通块的大小 $k$，有 $\frac{k(k + 1)}{2} \leq 2n \implies k \leq \sqrt{4n}$。

故直接考虑 dp，设 $f(u, i, 0 / 1)$ 表示考虑到了以 $u$ 为根的子树，$u$ 所在的连通块大小为 $i$ 颜色为 $0/1$ 时的最小负贡献。转移很简单 ...

时间复杂度 $\mathcal{O}(n\sqrt{n})$，但此题还卡了空间复杂度 $\mathcal{O}(n\sqrt{n})$ 的实现。

官方给出的解决方案是 [[Idea] Using HLD to reduce memory](https://codeforces.com/blog/entry/113453)。

18Michael 大佬的解决方案：使用 `std::vector` 来储存 dp 数组，每次枚举 $u$ 的子节点 $v$ 转移之后，将 $v$ 的 dp 数组进行 `clear()` 以及 `shrink_to_fit()`。**可以证明空间复杂度是 $\mathcal{O}(n)$ 的（容量不超过 $k$ 不重要，容量不超过子树大小比较重要）**。