---
title: 「CF1313D」Happy New Year
date: 2025-03-12 00:00:01
updated: 2026-03-04 00:00:01
categories: Codeforces
tags:
  - 「DP」
  - 状压 DP
---

# Description

Link：[CF1313D](https://codeforces.com/problemset/problem/1313/D)

{% note default %}

有 $m$ 个孩子，编号为 $1 \sim m$。

圣诞老人会 $n$ 种魔法，第 $i$ 种魔法可以给所有编号在区间 $[L_i, R_i]$ 内的孩子各发一个糖果。每种魔法最多使用一次，并且已知如果所有魔法都使用，每个孩子至多收到 $k$ 个糖果。

你可以控制这 $n$ 种魔法的使用情况，请你求出最多有多少个孩子收到奇数个糖果。

数据范围：$1 \leq n \leq 10^5$，$1 \leq m \leq 10^9$，$1 \leq k \leq 8$。

时空限制：$2$s / $500$MiB。

{% endnote %}

<!-- more -->

# Solution

注意到 $m$ 为 $10^9$ 级别，$n$ 为 $10^5$ 级别，而有效的区间个数不会超过 $2n + 1$ 个。

注意到 $k$ 非常小，考虑状态压缩，使用一个二进制数表示当前段的区间选取情况，在 dp 的过程中动态分配每个区间的编号。