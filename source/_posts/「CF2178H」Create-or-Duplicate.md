---
title: 「CF2178H」Create or Duplicate
date: 2026-01-16 22:31:52
updated: 2026-01-16 22:31:52
categories: Codeforces
tags:
  - 同余最短路
  - 状压 DP
---

# Description

Link：[CF2178H](https://codeforces.com/contest/2178/problem/H)

{% note default %}

有三种类型的礼物，价值分别为 $a, b, c$。初始时，恰好拥有每一种礼物各一份。

给出两个整数 $m$ 与 $k$。你可以进行若干次操作，每次操作形如以下的两种：
- 创造：选择一种礼物类型，并额外制作一份该类型的礼物。消耗 $x$ 点法力值，其中 $x \in \{ a, b, c \}$ 为所选礼物类型的价值。
- 复制：选择一种礼物类型，并复制该类型的所有礼物。消耗 $k$ 点法力值。

你需要求出，使得礼物价值总和是 $m$ 的倍数时，所需的最小法力值。

数据范围：$1 \leq a < b < c < m \leq 5 \times 10^5$，$1 \leq k \leq 5 \times 10^5$，$1 \leq \sum m, \sum k \leq 5 \times 10^5$。

时空限制：$6$s / $1024$MiB。

{% endnote %}

<!-- more -->

# Solution

首先三种类型的礼物是独立的。对于每种礼物而言，问题就转化为了简单的“同余最短路”。但由于 $(\min, +)$ 卷积只能做到 $\mathcal{O}(n^2)$，所以分开求解答案再合并的思路不太能做。

为什么不能只记录总价值 $\bmod \ m$ 作为状态？因为三种礼物混在一起，虽然创造操作仍然可以进行，但是复制操作我们就不知道如何更新状态了。

因此考虑挖掘操作的性质。注意到代价分别为 $k$ 的复制 $a\times 2, b\times 2$，相当于代价为 $2k$ 的复制 $(a + b) \times 2$。这启发我们，可以将复制操作捆绑在一起进行（而不是分开考虑它们）。

然而，由于初始时每个礼物各有一个，有时我们不希望复制某个元素。所以我们可以视作初态手上没有任何礼物，终态每个礼物必须被创造过（并且在最终的价值中减去 $a + b + c$）。

设状态 $(\mathrm{mask}, x)$ 表示当前已经创造礼物构成的二进制状态为 $\mathrm{mask}$，礼物总价值 $\bmod \ m$ 等于 $x$ 的情况。则我们有如下转移：
- $(\mathrm{mask}, x) \to (\mathrm{mask} \mid \mathtt{100}, (x + a) \bmod m)$，代价为 $a$。
- $(\mathrm{mask}, x) \to (\mathrm{mask} \mid \mathtt{010}, (x + b) \bmod m)$，代价为 $b$。
- $(\mathrm{mask}, x) \to (\mathrm{mask} \mid \mathtt{001}, (x + c) \bmod m)$，代价为 $c$。
- $(\mathrm{mask}, x) \to (\mathrm{mask}, 2x \bmod m)$，代价为 $|\mathrm{mask}| \cdot k$。

可以证明，任何一种操作序列，都等价于上述的状态转移。**简单证明：考虑先忽略所有的创造操作，将所有的复制操作倒序对齐，然后再将其他的创造操作插入进去**。

时间复杂度 $\mathcal{O}(m \log m)$。

```c++
#include <bits/stdc++.h>

using i64 = long long;

#define debug(a) std::cout << #a << '=' << (a) << ' '

template <class T>
inline void chmin(T &x, const T &y) {
    if (x > y) {
        x = y;
    }
}
template <class T>
inline void chmax(T &x, const T &y) {
    if (x < y) {
        x = y;
    }
}

const int N = 500100;
const i64 inf = 1e18;

int a, b, c, m, k;

i64 d[8][N];
int vis[8][N];

void work() {
    std::cin >> a >> b >> c >> m >> k;

    for (int mask = 0; mask < 8; mask ++) {
        for (int i = 0; i < m; i ++) {
            d[mask][i] = inf, vis[mask][i] = 0;
        }
    }
    d[0][0] = 0;

    std::priority_queue< std::tuple<i64, int, int> > q;
    q.push({0, 0, 0});

    while (q.size()) {
        auto [_, h, x] = q.top(); q.pop();

        if (vis[h][x]) {
            continue;
        }
        vis[h][x] = 1;

        auto extend = [&] (int nh, int nx, int w) {
            if (d[h][x] + w < d[nh][nx]) {
                d[nh][nx] = d[h][x] + w;
                q.push({-d[nh][nx], nh, nx});
            }
        };

        extend(h | (1 << 0), (x + a) % m, a);
        extend(h | (1 << 1), (x + b) % m, b);
        extend(h | (1 << 2), (x + c) % m, c);
        extend(h, 2 * x % m, __builtin_popcount(h) * k);
    }

    i64 ans = d[7][0] - a - b - c;
    std::cout << ans << '\n';
}

int main() {
    std::ios::sync_with_stdio(0);
    std::cin.tie(0);

    int T;
    std::cin >> T;

    while (T --) {
        work();
    }

    return 0;
}
```