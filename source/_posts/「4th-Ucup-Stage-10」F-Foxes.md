---
title: 「4th Ucup Stage 10」F. Foxes
date: 2026-01-15 20:03:39
updated: 2026-01-15 20:03:39
categories: Ucup
tags:
  - 「数据结构」
  - 线段树
---

# Description

Link：[QOJ 16005](https://qoj.ac/contest/2814/problem/16005)

{% note default %}

给出一个长度为 $n$ 的序列 $a_1, a_2, \dots, a_n$。有一个指针 $p$（初始时 $p = 1$）。

有 $Q$ 次操作，每次操作为以下三种之一：
- `<`：令 $p \gets p - 1$。
- `>`：令 $p \gets p + 1$。
- `! v`：令 $a_p \gets v$，求出整个序列的**严格最长上升子序列（LIS）**。

数据范围：$2 \leq n \leq 2\times 10^5$，$1 \leq Q \leq 5 \times 10^5$，$1\leq a_i, v \leq 10^6$。

时空限制：$2$s / $256$MiB。

{% endnote %}

<!-- more -->

# Solution

从这个题吸取了一点教训：以往某些题的经验实在不一定管用。因为之前做 [CF650D](https://codeforces.com/contest/650/problem/D) 的时候，并没有见到类似的做法，所以就以为不能这样维护。实际上这样做还是比较自然的，思维定式有点严重了。

设 $f_i$ 表示以 $i$ 为结尾的 LIS 长度，$g_i$ 表示以 $i$ 为开头的 LIS 长度。

则包含 $p$ 的全局 LIS 长度为 $f_p + g_p - 1$，不包含 $p$ 的全局 LIS 长度为 $\max\limits_{x < p < y, a_x < a_y} \{f_x + g_y\}$。

由于每次指针 $p$ 移动的步长为 $1$，所以我们可以**动态地将 $f_{[1, p - 1]}$ 以及 $g_{[p + 1, n]}$ 维护出来**。虽然每次修改 $a_p$ 会影响所有的 $g_{[1, p - 1]}$ 与 $f_{[p + 1, n]}$，但在求解答案的时候，我们只关心 $f_{[1, p - 1]}$ 和 $g_{[p + 1, n]}$。所以我们只需在移动指针的过程中，顺带求出新的 $f_p, g_p$ 即可（没有遍历到的点暂且不会对答案产生影响）。

回到如何求解答案，我们开一个值域线段树维护 $f_{[1, p - 1]}$ 与 $g_{[p + 1, n]}$（以 $a_i$ 为下标）。

包含 $p$ 的全局 LIS 长度 $f_p + g_p - 1$ 是好求的。

不包含 $p$ 的全局 LIS 长度 $\max\limits_{a_x < a_y} \{ f_x + g_y \}$ 可以在线段树上传信息的时候顺带求出。请注意这里我们动态维护出了 $f_{[1, p - 1]}$ 与 $g_{[p + 1, n]}$，所以自然可以去掉 $x < p < y$ 的限制。

时间复杂度 $\mathcal{O}((n + Q) \log n)$。

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

const int N = 200100;

const int sup = 1000001;
const int SIZE = 1000100;

int n, Q;
int a[N];

int f[N], g[N];

std::multiset<int> fv[SIZE], gv[SIZE];

namespace SGT {
    struct node {
        int f, g;
        int ans;
    } t[SIZE * 4];

    void upd(int p) {
        t[p].f = std::max(t[p * 2].f, t[p * 2 + 1].f);
        t[p].g = std::max(t[p * 2].g, t[p * 2 + 1].g);
        t[p].ans = std::max(std::max(t[p * 2].ans, t[p * 2 + 1].ans), t[p * 2].f + t[p * 2 + 1].g);
    }

    void change(int p, int l, int r, int x, int y, int opt, int type) {
        if (l == r) {
            if (type == 1) {
                opt == +1 ? fv[l].insert(y) : fv[l].erase(fv[l].find(y));
                t[p].f = fv[l].empty() ? 0 : *(-- fv[l].end());
            } else {
                opt == +1 ? gv[l].insert(y) : gv[l].erase(gv[l].find(y));
                t[p].g = gv[l].empty() ? 0 : *(-- gv[l].end());
            }
            t[p].ans = t[p].f + t[p].g;
            return;
        }
        int mid = (l + r) >> 1;
        if (x <= mid) {
            change(p * 2, l, mid, x, y, opt, type);
        } else {
            change(p * 2 + 1, mid + 1, r, x, y, opt, type);
        }
        upd(p);
    }
    void change(int x, int y, int opt, int type) {
        change(1, 0, sup, x, y, opt, type);
    }

    int ask(int p, int l, int r, int s, int e, int type) {
        if (s <= l && r <= e) {
            return type == 1 ? t[p].f : t[p].g;
        }
        int mid = (l + r) >> 1;
        if (s <= mid && mid < e) {
            return std::max(ask(p * 2, l, mid, s, e, type), ask(p * 2 + 1, mid + 1, r, s, e, type));
        }
        if (s <= mid) {
            return ask(p * 2, l, mid, s, e, type);
        } else {
            return ask(p * 2 + 1, mid + 1, r, s, e, type);
        }
    }
    int ask(int s, int e, int type) {
        return ask(1, 0, sup, s, e, type);
    }
}

int main() {
    std::ios::sync_with_stdio(0);
    std::cin.tie(0);

    std::cin >> n >> Q;

    for (int i = 1; i <= n; i ++) {
        std::cin >> a[i];
    }

    for (int i = n; i >= 2; i --) {
        g[i] = SGT::ask(a[i] + 1, sup, 2) + 1;
        SGT::change(a[i], g[i], +1, 2);
    }
    f[1] = 1;

    int p = 1;
    while (Q --) {
        std::string str;
        std::cin >> str;

        if (str == ">") {
            SGT::change(a[p], f[p], +1, 1);
            SGT::change(a[p + 1], g[p + 1], -1, 2); 
            p ++;
            f[p] = SGT::ask(0, a[p] - 1, 1) + 1;
        } else if (str == "<") {
            SGT::change(a[p - 1], f[p - 1], -1, 1);
            SGT::change(a[p], g[p], +1, 2);
            p --;
            g[p] = SGT::ask(a[p] + 1, sup, 2) + 1;
        } else {
            int y;
            std::cin >> y;
            
            a[p] = y;
            f[p] = SGT::ask(0, y - 1, 1) + 1;
            g[p] = SGT::ask(y + 1, sup, 2) + 1;

            int ans1 = f[p] + g[p] - 1;
            int ans2 = SGT::t[1].ans;

            std::cout << std::max(ans1, ans2) << '\n';
        }
    }

    return 0;
}

/*
5 2
3 4 1 7 2
>
! 2

5 8
3 4 1 7 2
>
! 2
>
>
>
! 10
<
! 11
*/
```