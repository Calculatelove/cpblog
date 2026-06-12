---
title: 「gym103483」C. How Many Strings Are Less
date: 2025-03-25 00:00:01
updated: 2026-03-05 16:28:55
categories: Codeforces
tags:
  - Trie
---

# Description

Link：[gym103483 C](https://codeforces.com/gym/103483/problem/C)

{% note default %}

给出一个包含 $n$ 个字符串的集合 $D$ 以及一个字符串 $s$，你需要找出集合 $D$ 中字典序小于 $s$ 的字符串数量。

字符串 $s$ 会经过 $Q$ 次修改，每次修改会给出一个整数 $k$ 以及一个字符 $c$，表示将字符串 $s$ 从第 $k$ 个字符开始到字符串末尾的所有字符替换成 $c$。每次修改完字符串 $s$ 之后，你都需要求出集合 $D$ 中字典序小于 $s$ 的字符串数量。

数据范围：$1 \leq n, Q \leq 10^6$，$1 \leq |s| \leq 10^6$，字符串集合 $D$ 的总长不超过 $10^6$。

时空限制：$2$s / $512$MiB。

{% endnote %}

<!-- more -->

# Solution

考虑建出 Trie 树，设 $\mathrm{cnt}[u]$ 表示节点 $u$ 上的终止节点个数，设 $\mathrm{sze}[u]$ 表示节点 $u$ 的子树内的终止节点个数。

设 $\mathrm{go}(u, c)$ 表示节点 $u$ 往节点 $\delta(u, c)$ 走产生的贡献，则 $\mathrm{go}(u, c) = \mathrm{cnt}[u] + \sum_{i < c} \mathrm{sze}[\delta(u, i)]$。

考虑把贡献记在点上，即 $\mathrm{value}(\delta(u, c)) = \mathrm{go}(u, c)$，则一个字符串 $s$ 对应的答案即为其在 Trie 树上经过的节点的贡献之和（叶子需要特殊考虑）。

于是我们只需维护字符串 $s$ 在 Trie 树上的位置即可。维护 $\mathrm{end}(u, c)$ 表示节点 $u$ 一直沿着字符 $c$ 走到达的节点。使用 $\mathrm{end}$ 数组向下走，再使用树上倍增向上走。特别处理一下叶子节点的贡献（即字符串 $s$ 走出了 Trie 树）即可。

MENASQ_TGRM 大佬的优秀做法：在 $\mathrm{end}(u, c)$ 中约束向下走到达的节点深度，深度 $\leq n$ 即可。