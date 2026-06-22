---
title: 「CF2233E2」Permutation Transmission (Difficult Version)
date: 2026-06-22 10:38:25
updated: 2026-06-22 10:38:25
categories: Codeforces
tags:
  - 结论
---

# Description

Link：[CF2233E2](https://codeforces.com/contest/2233/problem/E2)

{% note default %}

对于一个长度为 $n$ 的排列 $p$，记 $k = \lceil \log_2(n + 1) \rceil$。

系统原本会生成 $k$ 个长度为 $n$ 的 $01$ 串，对于第 $j$ ($0 \leq j < k$) 个二进制位，$01$ 串的第 $i$ 个字符表示 $p_i$ 的第 $j$ 个二进制位的值。

现在，这 $k$ 个 $01$ 串被打乱，因此不知道每个 $01$ 串对应哪个二进制位。

给定打乱后的 $k$ 个 $01$ 串，求有多少个不同的关于 $n$ 的排列 $p$，使得至少存在一种 $01$ 串与二进制位的对应关系，能够还原出 $p$。

数据范围：$1 \leq n \leq 2\times 10^5$。

时空限制：$2$s / $512$MiB。

{% endnote %}

<!-- more -->

# Solution

首先可以做一些简单的判断：
- 不存在某一列的串均为 $0$，这样就会使得 $p$ 中存在 $0$。
- 不存在某两列的串相等，这样就会使得 $p$ 中存在两个相同的数。

那么可以保证，不论串和二进制位的对应关系是怎么样，$p$ 一定是由 $n$ 个互不相同的正整数构成。

在该种情况下，**当 $p$ 是关于 $n$ 的排列的时候，$\sum p$ 一定是最小的**。反过来也成立。

所以我们现在需要找到一种串和二进制位的对应关系，使得 $\sum p$ 最小。假设第 $i$ 个 $01$ 串一共有 $c_i$ 个 $1$，那么将其对应到第 $j$ 个二进制位产生的贡献为 $c_i \times 2^j$。

所以要想使得 $\sum p$ 最小，只需将所有 $01$ 串按照 $1$ 的个数降序排序即可。降序排序后，先检查一下还原的排列是否为关于 $n$ 的排列。

此时还要统计方案数，不难发现 $1$ 的个数相同的两个串可以交换位置，所以设 $\mathrm{cnt}_i$ 表示有多少个 $01$ 串 $1$ 的个数等于 $i$，答案即为 $\prod_{i = 1}^n \mathrm{cnt}_i !$。

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

// 精确取对数（上取整）：找到最小整数 t，使得 a^t >= b
int iLog(i64 a, i64 b) {
    int t = 0;
    i64 v = 1;
    while (v < b) {
        t ++; if (v > b / a) break;
        v *= a;
    }
    return t;
}

const int N = 200100;

int n, k;
std::string str[20];
int one[20];
int ord[20];

int p[N];

int buc[N];
i64 fact[20];

void work() {
    std::cin >> n, k = iLog(2, n + 1);

    for (int j = 0; j < k; j ++) {
        std::cin >> str[j];
        str[j] = " " + str[j];

        one[j] = 0;
        for (int i = 1; i <= n; i ++) {
            one[j] += str[j][i] == '1';
        }
    }

    for (int j = 0; j < k; j ++) {
        ord[j] = j;
    }
    std::sort(ord, ord + k, [&] (int x, int y) -> bool {
        return one[x] > one[y];
    });

    for (int i = 1; i <= n; i ++) {
        p[i] = 0;
    }
    for (int j = 0; j < k; j ++) {
        for (int i = 1; i <= n; i ++) {
            if (str[ord[j]][i] == '1') {
                p[i] += (1 << j);
            }
        }
    }
    
    std::sort(p + 1, p + 1 + n);
    for (int i = 1; i <= n; i ++) {
        if (p[i] != i) {
            std::cout << 0 << '\n';
            return;
        }
    }

    fact[0] = 1;
    for (int i = 1; i <= k; i ++) {
        fact[i] = fact[i - 1] * i;
    }

    for (int i = 1; i <= n; i ++) {
        buc[i] = 0;
    }
    for (int j = 0; j < k; j ++) {
        buc[one[j]] ++;
    }
    
    i64 ans = 1;
    for (int i = 1; i <= n; i ++) {
        ans *= fact[buc[i]];
    }
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