---
title: 「2025 ICPC 网络赛 2」L. Xor Mirror
date: 2025-09-15 19:11:24
updated: 2025-09-15 19:11:24
categories: ICPC
tags:
  - 分块
---

# Description

Link：[QOJ 14325](https://qoj.ac/contest/2524/problem/14325)

{% note default %}

给出一个长度为 $n$ 的序列 $a_0, a_1, \dots, a_{n - 1}$，**保证 $n$ 是 $2$ 的若干次幂**。

有 $Q$ 次操作，每次操作都形如以下操作中的一种：
- `1 l r k`：对于所有 $i \in [l, r)$，令 $b_i = a_{i \oplus k}$。然后对于所有 $i \in [l, r)$，令 $a_i = b_i$。
- `2 l r`：求 $\sum_{i = l}^{r - 1} a_i$ 的值。

其中 $\oplus$ 表示按位异或。

数据范围：$1 \leq n \leq 2^{18}$，保证 $n$ 是 $2$ 的若干次幂，$1 \leq Q \leq 2 \times 10^5$，$1 \leq a_i \leq 1048576$，$0 \leq l < r \leq n$，$0 \leq k < n$。

时空限制：$2$s / $64$MiB。

{% endnote %}

<!-- more -->

# Solution

新学的一种分块方式：**高低位分块**。

设 $n = 2^{P}$，则取块长 $B = 2^{\left\lceil \frac{P}{2} \right\rceil}$。对于一个位置 $i$，将其分解成 $i = xB + y$ 的形式，**其实这里的 $x$ 和 $y$ 就分别代表了高位和低位，显然高位和低位分别进行位运算是互不影响的**。

建立一个包含 $\frac{n}{B}$ 个长度为 $B$ 的辅助块数组，用于存放不同种类块的内部结构（**可以理解为信息库，用于代表原序列中的若干个块，原序列中可能有许多个块指向同一个辅助块**）。

在原序列中，对于每个长度为 $B$ 的块 $i$，其高位都是一样的。我们使用标记 $(\mathrm{tagx}_i, \mathrm{tagy}_i)$ 表示：表示当前块在辅助块数组中的编号为 $\mathrm{tagx}_i$，并且下标的低位需要异或上 $\mathrm{tagy}_i$。相当于是通过标记，将“原序列的块”与“块数组”之间建立了联系。

询问操作是朴素的，整块预处理总和 + 散块暴力统计即可。

修改整块，也是好做的。对于原序列中的块 $i$，设 $k = xB + y$，则令 $\mathrm{tagx}'_i \gets \mathrm{tagx}_{i \oplus x}$，$\mathrm{tagy}'_i \gets \mathrm{tagy}_{i \oplus x} \oplus y$。

修改散块，可以考虑将散块暴力重构，然后再放入辅助块数组中，并且更新散块的 $\mathrm{tagx}$ 与 $\mathrm{tagy}(\mathrm{tagy} = 0)$。**注意这里我们无需在辅助块数组中新建节点，如果一个辅助块没有被任何一个原序列中的块指向，我们就可以将其删掉**。显然原序列中的 $\frac{n}{B} - 1$ 个块，最多产生 $\frac{n}{B} - 1$ 个被指向的辅助块，所以此时一定存在一个空位可以让新块塞入。

时间复杂度 $\mathcal{O}(Q(B + \frac{n}{B}))$，空间复杂度 $\mathcal{O}(n)$。

```c++
#include <bits/stdc++.h>

using s64 = long long;

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

int Q;
int n, pn;
int b, bn;

int X, Y;

std::vector< std::vector<int> > seq;
std::vector<s64> sum;

std::vector<int> tagx, tagy, tmpx, tmpy;
std::vector<int> hold;

int pos1, pos2;
void getPos() {
    pos1 = pos2 = -1;
    for (int i = 0; i < X; i ++) {
        if (hold[i] == 0) {
            if (pos1 == -1) {
                pos1 = i;
            } else if (pos2 == -1) {
                pos2 = i;
            } else {
                return;
            }
        }
    }
}

void change(int l, int r, int k) {
    int lx = l / b, ly = l % b;
    int rx = r / b, ry = r % b;
    int kx = k / b, ky = k % b;

    if (lx == rx) {
        hold[tagx[lx]] --;

        s64 s = 0;
        std::vector<int> num(Y);

        for (int j = 0; j < Y; j ++) {
            if (j < ly || j > ry) {
                num[j] = seq[tagx[lx]][j ^ tagy[lx]];
            } else {
                num[j] = seq[tagx[lx ^ kx]][j ^ tagy[lx ^ kx] ^ ky];
            }
            s += num[j];
        }

        getPos();

        tagx[lx] = pos1, tagy[lx] = 0, hold[pos1] ++;
        seq[pos1] = num, sum[pos1] = s;
    } else {
        hold[tagx[lx]] --, hold[tagx[rx]] --;

        s64 s1 = 0, s2 = 0;
        std::vector<int> num1(Y), num2(Y);

        for (int j = 0; j < Y; j ++) {
            if (j < ly) {
                num1[j] = seq[tagx[lx]][j ^ tagy[lx]];
            } else {
                num1[j] = seq[tagx[lx ^ kx]][j ^ tagy[lx ^ kx] ^ ky];
            }
            s1 += num1[j];
        }

        for (int j = 0; j < Y; j ++) {
            if (j > ry) {
                num2[j] = seq[tagx[rx]][j ^ tagy[rx]];
            } else {
                num2[j] = seq[tagx[rx ^ kx]][j ^ tagy[rx ^ kx] ^ ky];
            }
            s2 += num2[j];
        }

        if (rx - lx > 1) {
            tmpx = tagx, tmpy = tagy;
            for (int i = lx + 1; i <= rx - 1; i ++) {
                hold[tagx[i]] --, hold[tmpx[i ^ kx]] ++;

                tagx[i] = tmpx[i ^ kx];
                tagy[i] = tmpy[i ^ kx] ^ ky;
            }
        }

        getPos();

        // for (int x : num1) {
        //     std::cout << x << ' ';
        // }
        // for (int x : num2) {
        //     std::cout << x << ' ';
        // }
        // std::cout << '\n';

        tagx[lx] = pos1, tagy[lx] = 0, hold[pos1] ++;
        seq[pos1] = num1, sum[pos1] = s1;

        tagx[rx] = pos2, tagy[rx] = 0, hold[pos2] ++;
        seq[pos2] = num2, sum[pos2] = s2;
    }
}

s64 ask(int l, int r) {
    int lx = l / b, ly = l % b;
    int rx = r / b, ry = r % b;

    s64 ans = 0;
    if (lx == rx) {
        for (int j = ly; j <= ry; j ++) {
            ans += seq[tagx[lx]][j ^ tagy[lx]];
        }
    } else {
        for (int j = ly; j < Y; j ++) {
            ans += seq[tagx[lx]][j ^ tagy[lx]];
        }
        for (int i = lx + 1; i <= rx - 1; i ++) {
            ans += sum[tagx[i]];
        }
        for (int j = 0; j <= ry; j ++) {
            ans += seq[tagx[rx]][j ^ tagy[rx]];
        }
    }
    return ans;
}

void work() {
    std::cin >> n >> Q;

    pn = 0;
    while ((1 << pn) < n) pn ++;

    bn = (pn + 1) / 2, b = (1 << bn);

    X = n / b, Y = b;

    seq.resize(X);
    sum.resize(X);

    tagx.resize(X), tagy.resize(X);
    hold.resize(X);

    for (int i = 0; i < X; i ++) {
        sum[i] = 0;
        tagx[i] = i, tagy[i] = 0, hold[i] = 1;
    }

    for (int i = 0; i < X; i ++) {
        std::vector<int> num(Y);
        for (int j = 0; j < Y; j ++) {
            int v;
            std::cin >> v;
            num[j] = v, sum[i] += v;
        }
        seq[i] = num;
    }

    while (Q --) {
        int opt, l, r, k;
        std::cin >> opt >> l >> r, r --;

        if (opt == 1) {
            std::cin >> k;
            change(l, r, k);
        } else {
            std::cout << ask(l, r) << '\n';
        }
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

/*
1
8 2
7 3 8 1 4 6 4 1
2 2 7
2 5 7

1
8 4
7 3 8 1 4 6 4 1
2 7 8
1 3 5 3
1 5 6 2
2 7 8

1
8 8
7 3 8 1 4 6 4 1
2 2 7
2 5 7
1 3 5 3
1 5 6 2
2 7 8
2 3 7
1 2 8 5
2 5 8
*/
```