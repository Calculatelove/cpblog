---
title: 「CF1096E」The Top Scorer
date: 2025-03-16 00:00:02
updated: 2026-03-04 00:00:02
categories: Codeforces
tags:
  - 概率
  - 组合数学
  - 容斥
---

# Description

Link：[CF1096E](https://codeforces.com/contest/1096/problem/E)

{% note default %}

有 $p$ 个人玩一个游戏，第 $i$ 个人的得分为 $a_i$。

已知 $\sum a_i = s$ 且 $a_1 \geq r$，得分最高的人可以获胜，若多个人得分最高，则等概率随机其中一个人获胜。

求第一个人获胜的概率，答案对 $998244353$ 取模。

数据范围：$1 \leq p \leq 100$，$0 \leq r \leq s \leq 5000$。

时空限制：$3$s / $250$MiB。

{% endnote %}

<!-- more -->

# Solution

naive：枚举最大值 $\mathrm{mx}$，枚举最大值个数 $c$，计算剩余选手得分 $< \mathrm{mx}$ 且和等于 $s - \mathrm{mx} \cdot c$ 的方案数。

不妨去掉小明得分 $\geq r$ 的限制，此时每个人成为冠军的概率均等，均为 $\frac{1}{n}$。

“小明得分 $\geq r$ 且小明是冠军” 等价于 “冠军得分 $\geq r$ 且小明是冠军”，故我们只需计算冠军得分 $\geq r$ 的概率，最后乘以 $\frac{1}{n}$ 即可。

考虑容斥，钦定 $i$ 个选手得分 $\geq r$，先将这 $i$ 个选手的得分减去 $r$，故我们需要计算选手得分总和为 $s - i \cdot r$ 的方案数，运用插板法可得

$$
\sum\limits_{i = 1}^n (-1)^{i + 1} \binom{n}{i} \binom{s - i \cdot r + n - 1}{n - 1}
$$