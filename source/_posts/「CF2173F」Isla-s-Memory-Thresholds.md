---
title: 「CF2173F」Isla's Memory Thresholds
date: 2025-12-08 16:05:58
updated: 2025-12-08 16:05:58
categories: Codeforces
tags:
  - 平衡规划
---

# Description

Link：[CF2173F](https://codeforces.com/contest/2173/problem/F)

{% note default %}

给出一个长度为 $n$ 的**非递增序列** $a_1, a_2, \cdots, a_n$（满足 $a_1 \geq a_2 \geq \cdots \geq a_n$）。

有 $Q$ 次查询，每次查询给出 $l, r, x$，你会依次遍历 $a_l, a_{l + 1}, \cdots, a_r$，你有一个计数器 $s$（初始 $s = 0$），每次将当前遍历到的数累加进 $s$，若任意时刻 $s \geq x$，你就会将 $s$ 清零。你需要求出遍历结束时，将 $s$ 清零的次数以及最后 $s$ 的值。

数据范围：$1 \leq n, Q \leq 1.5 \times 10^5$，$1 \leq a_i, x \leq 10^9$，$1 \leq l \leq r \leq n$。

时空限制：$6$s / $1024$MiB。

{% endnote %}

<!-- more -->

# Solution

## 算法一

在询问过程中，记 $s$ 从零到超出阈值 $x$ 的区间为 **“整段”**。由于序列 $a$ 非递增，那么询问过程中，**依次经过的整段长度必定是非递减的**。

考虑根号分治。记阈值 $B$：
- 对于长度 $\leq B$ 的整段。这样的长度仅有 $\mathcal{O}(B)$ 个，对于相同长度的整段我们放在一起处理，可以二分找到允许当前长度跳跃的**最远右端点**，然后一次性跳完当前长度的整段即可。
- 对于长度 $>B$ 的整段。至多只需要跳 $\mathcal{O}(\frac{n}{B})$ 次，于是二分找到**当前整段的右端点**，暴力跳整段即可。

取 $B = \sqrt{n}$，时间复杂度 $\mathcal{O}(n \sqrt{n} \log n)$。

## 算法二

进一步，发现长度 $\leq B$ 的整段的跳跃可以预处理。

具体地，离线将所有询问的阈值 $x$ 记录下来。

需要预处理出 `f[len][t]` 表示：当阈值为询问的第 $t$ 小的阈值时，最后一个长度为 $\mathrm{len}$ 且总和超过阈值的区间右端点（实际上这就是允许长度为 $\mathrm{len}$ 跳跃的最远右端点）。

对于所有长度为 $\mathrm{len}$ 的区间来说，位置越靠前总和越大。所以我们直接从右往左依次扫描所有长度为 $\mathrm{len}$ 的区间，记当前总和已经超过第 $t$ 小的阈值。每次向左扩展时，就不断地判断当前总和是否超过第 $t + 1$ 小的阈值即可。

取 $B = \sqrt{n \log n}$，时间复杂度 $\mathcal{O}(n \sqrt{n \log n})$。

```c++
#include <bits/stdc++.h>

using i64 = long long;
using u64 = unsigned long long;

#define debug(a) std::cout << #a << "=" << (a) << ' '

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

const int N = 150100, MaxQ = 150100;
const int B = 520;

int n, Q;
int a[N];

i64 pre[N];

int amo, mapval[N]; // amo: amount
int turn(int x) {
    return std::lower_bound(mapval + 1, mapval + 1 + amo, x) - mapval;
}

std::tuple<int, int, int> qry[MaxQ];

void work() {
    std::cin >> n >> Q;

    for (int i = 1; i <= n; i ++) {
        std::cin >> a[i];
        pre[i] = pre[i - 1] + a[i];
    }

    amo = 0;
    for (int i = 1; i <= Q; i ++) {
        int l, r, v;
        std::cin >> l >> r >> v;

        mapval[++ amo] = v;
        qry[i] = {l, r, v};
    }

    std::sort(mapval + 1, mapval + 1 + amo);
    amo = std::unique(mapval + 1, mapval + 1 + amo) - mapval - 1;

    std::vector< std::vector<int> > f(B + 1, std::vector<int>(amo + 1, 0));
    for (int len = 1; len <= B; len ++) {
        for (int x = n, t = 0; x >= len; x --) {
            i64 sum = pre[x] - pre[x - len];
            while (t < amo && sum >= mapval[t + 1]) {
                t ++;
                f[len][t] = x;
            }
        }
    }

    for (int qid = 1; qid <= Q; qid ++) {
        auto [l, r, v] = qry[qid];
        int id = turn(v);

        int p = l, res = 0;
        for (int len = 1; len <= B; len ++) {
            int rpos = f[len][id];
            if (rpos >= p) {
                int step = (std::min(r, rpos) - p + 1) / len;
                p += step * len;
                res += step;
            }
        }

        while (pre[r] - pre[p - 1] >= v) {
            int l1 = p, r1 = r;
            while (l1 < r1) {
                int mid = (l1 + r1) >> 1;
                if (pre[mid] - pre[p - 1] >= v) {
                    r1 = mid;
                } else {
                    l1 = mid + 1;
                }
            }

            p = l1 + 1;
            res ++;
        }

        std::cout << res << ' ' << (pre[r] - pre[p - 1]) << '\n';
    }
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

/**
 *  心中无女人
 *  比赛自然神
 *  模板第一页
 *  忘掉心上人
**/
```