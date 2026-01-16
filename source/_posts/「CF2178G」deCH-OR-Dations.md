---
title: 「CF2178G」deCH OR Dations
date: 2026-01-16 20:59:59
updated: 2026-01-16 20:59:59
categories: Codeforces
tags:
  - 异或哈希
  - 树状数组
---

# Description

Link：[CF2178G](https://codeforces.com/contest/2178/problem/G)

{% note default %}

圆环上有 $2n$ 个等间距的点，按顺时针顺序标记为 $1, 2, \cdots, 2n$。有 $n$ 条具有不同端点的弦，其中第 $i$ 条弦连接 $a_i, b_i$。现在按照顺序依次连接这 $n$ 条弦。

每当连接完前 $\ell$ 条弦以后，考虑这 $\ell$ 条弦的任意非空子集 $S$。设 $S$ 中元素的索引分别为 $c_1, c_2, \cdots, c_m$，若对于所有的 $1 \leq i < m$，都满足弦 $c_i$ 与弦 $c_{i + 1}$ 相交，则称 $S$ 为一条链。

当且仅当，每一条弦在前 $\ell$ 条弦的所有子集链中出现偶数次时，称这些弦是紧密连接的。

对于 $1$ 到 $n$ 的每一个 $\ell$，你都需要判断前 $\ell$ 条弦是否紧密连接。

数据范围：$2 \leq n \leq 5 \times 10^5$，$1 \leq a_i < b_i \leq 2n$。

时空限制：$3$s / $512$MiB。

{% endnote %}

<!-- more -->

# Solution

设 $f_i$ 表示考虑了前 $i$ 条弦，以弦 $i$ 为结尾的链个数。由于我们只关心链个数的奇偶性，所以我们只需维护 $\bmod 2$ 意义下的值即可。显然有转移

$$
f_i = \left(\bigoplus_{j < i, \mathrm{ok}(j, i) = 1} f_j\right) \oplus 1
$$

其中 $\mathrm{ok}(j, i)$ 表示弦 $j$ 与弦 $i$ 是否相交。

考虑维护 $f$。由于异或有着出现偶数次相消的性质，维护一个树状数组，其中 $a_j$ 与 $b_j$ 处均储存着 $f_j$ 的值，那么转移的式子就只需要查询区间 $[a_i, b_i]$ 内的异或和即可。**因为任何与弦 $i$ 不相交的弦 $j$，贡献只会计入 $0$ 次或 $2$ 次**。

更进一步，设 $g(i, j)$ 表示考虑了前 $i$ 条弦，以弦 $i$ 为结尾且包含弦 $j$ 的情况下，链个数的奇偶性。

设 $h_i$ 表示集合 $\{ j : g(i, j) = 1 \}$，显然有转移（这里的异或 $\oplus$ 是针对集合的）

$$
h_i = \left(\bigoplus_{j < i, \mathrm{ok}(j, i) = 1} h_j\right) \oplus [f_i = 1]\{ i \}
$$

所以判断前 $\ell$ 条弦是否紧密相连，只需要判断 $\oplus_{i = 1}^{\ell} h_i$ 是否为空集即可。

现在要考虑快速维护 $h$。由于我们只需要判断集合是否为空集，可以考虑使用**异或哈希**！具体地，对 $1, \cdots, n$ 中的每一个数映射到一个 $64$ 位大整数。**对于一个集合来说，该集合的哈希值即为集合所有元素映射值的异或和**。那么两个集合异或后的哈希值即为各自哈希值的异或。于是我们就可以沿用上述维护 $f$ 的方法来维护 $h$。

时间复杂度 $\mathcal{O}(n \log n)$。

```c++
#include <bits/stdc++.h>

using i64 = long long;

using u64 = unsigned long long;

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

int n;

struct BIT {
    int n;
    std::vector<u64> c;

    BIT() {}
    BIT(int _n) : n(_n) {
        c.assign(n + 1, {});
    }

    void add(int x, u64 y) {
        for (; x <= n; x += x & -x) {
            c[x] ^= y;
        }
    }

    u64 ask(int x) {
        u64 ans = 0;
        for (; x; x -= x & -x) {
            ans ^= c[x];
        }
        return ans;
    }
};

std::mt19937_64 mtrand{std::random_device{}()};

void work() {
    std::cin >> n;

    BIT b1(n * 2), b2(n * 2);

    std::string ans;
    u64 H = 0;
    for (int i = 1; i <= n; i ++) {
        int x, y;
        std::cin >> x >> y;

        u64 f = b1.ask(x) ^ b1.ask(y) ^ 1;
        u64 h = b2.ask(x) ^ b2.ask(y);
        if (f) {
            h ^= mtrand();
        }

        H ^= h;
        ans += H ? "0" : "1";

        b1.add(x, f), b1.add(y, f);
        b2.add(x, h), b2.add(y, h);
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