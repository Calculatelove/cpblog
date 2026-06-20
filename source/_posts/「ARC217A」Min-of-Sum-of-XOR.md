---
title: 「ARC217A」Min of Sum of XOR
date: 2026-06-20 17:07:38
updated: 2026-06-20 17:07:38
categories: AtCoder
tags:
  - 构造
---

# Description

Link：[ARC217A](https://atcoder.jp/contests/arc217/tasks/arc217_a)

{% note default %}

给出一个正整数 $n$。

构造一个关于 $n$ 的排列，使得 $\sum_{i = 1}^n \bigoplus_{j = 1}^i p_j$ 最小。

其中 $\bigoplus_{j = 1}^i p_j$ 表示 $p_1, \dots, p_i$ 的异或和。

数据范围：$1 \leq n \leq 2\times 10^5$。

时空限制：$2$s / $1024$MiB。

{% endnote %}

<!-- more -->

# Solution

首先要考虑这个问题的下界。

不妨先思考一个子问题：对于一个长度为 $n$ 的 $01$ 序列，如何重排才可以使得前缀异或和之和最小？
- 假设一共有 $c$ 个 $1$，应该将第 $2i - 1$ 个 $1$ 与第 $2i$ 个 $1$ 相邻摆放。特别地，若 $c$ 为奇数，则最后一个 $1$ 需要放在第 $n$ 位上。

继续考虑这个问题的下界。对于每一个二进制位，都要遵循上述的摆放规则，才可以取到下界。

回顾一个关键性质：**$4i, 4i + 1, 4i + 2, 4i + 3$ 的异或和为 $0$**。

并且 $4i, 4i + 1, 4i + 2, 4i + 3$ 的所有第 $k$ ($k \geq 2$) 个二进制位全都相等。这启发我们，以 $4$ 为步长进行分段。

不难发现对于排列 $1, \dots, n$，只有第 $0, 1$ 个二进制位还不满足要求。

**将分段调整成 $4i, 4i + 1, 4i + 3, 4i + 2$**。这样第 $0$ 个二进制位和第 $1$ 个二进制位段内分别为 $(0, 1, 1, 0), (0, 0, 1, 1)$，即可符合要求。

对于最后不足 $4$ 个的一段，就直接按顺序填写剩下的数即可。

注意到当 $n \equiv 0, 1, 3 \pmod 4$ 时，显然也可以取到下界。只有 $n \equiv 2 \pmod 4$ 这一例外，此时最后一段是不完整的。

**事实上 $n \equiv 2 \pmod 4$ 的时候是取不到下界的**。假设存在某个排列 $p$ 取到了下界：
- 因为 $1, \dots, n - 1$ 中所有第 $k$ ($k \geq 1$) 个二进制位都有偶数个，此时再将 $n$ 加入的话，根据规则就必然有 $p_n = n$。
- 但是此时 $1, \dots, n$ 中的第 $0$ 个二进制位有奇数个，若想取到下界必有 $p_n$ 为奇数，这与 $p_n = n$ 矛盾。

但该种情况经过上述构造可以取到下界 $+1$，所以该种构造是正确的。

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

void work() {
    int n;
    std::cin >> n;

    std::vector<int> ans(n + 1);
    std::iota(ans.begin(), ans.end(), 0);

    for (int i = 3; i <= n; i += 4) {
        std::swap(ans[i - 1], ans[i]);
    }
    for (int i = 1; i <= n; i ++) {
        std::cout << ans[i] << " \n"[i == n];
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
```

