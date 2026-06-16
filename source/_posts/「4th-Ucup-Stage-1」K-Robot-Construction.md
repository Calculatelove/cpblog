---
title: 「4th Ucup Stage 1」K. Robot Construction
date: 2025-10-06 20:50:35
updated: 2025-10-06 20:50:35
categories: Ucup
tags:
  - 线段树
---

# Description

Link：[QOJ 14436](https://qoj.ac/contest/2539/problem/14436)

{% note default %}

给出一个非负整数序列 $a_1, a_2, \dots, a_n$。初始时，你可以创造一个机器人，高度为 $[0, d]$ 之间的正整数。每当经过一个检查点 $a_i$ 时，如果当前的高度 $h$ 满足 $h \geq a_i$，则高度 $h \gets h - a_i$；否则不会发生任何事。

有 $Q$ 次询问，每次询问给出两个正整数 $l, r$ ($1 \leq l \leq r \leq n$)，你需要选择机器人的初始高度，使得依次经过检查点 $a_l, \dots, a_r$ 之后机器人剩下的高度最大，你只需要求出该高度即可。

数据范围：$1 \leq n, Q \leq 3 \times 10^5$，$1 \leq d \leq 10^9$，$0 \leq a_i \leq 10^9$，$1 \leq l \leq r \leq n$。

时空限制：$1$s / $256$MiB。

{% endnote %}

<!-- more -->

# Solution

有一些基本的推论。设 $f(x)$ 表示初始高度为 $x$ 的机器人，可以发现 $f(x)$ 每时每刻都是由若干个以 $0$ 开始斜率为 $1$ 的线段组成。每次经过一个检查点 $a_i$ 时，对于一个最高点为 $h$ 的段，若 $h \geq a_i$，则会被拆成两个最高点分别为 $a_i - 1$ 与 $h - a_i$ 的段。

进一步，可以发现**只需维护最高的段即可**。因为越大的段，经过一个检查点后生成的段也越大。

设 $x$ 表示最高高度，经过一个检查点 $a_i$ 时
- 若 $x \in [0, a_i)$ 时，$x$ 不变。
- 若 $x \in [a_i, 2a_i)$ 时，$x \gets a_i - 1$。
- 若 $x \in [2a_i, +\infty)$ 时，$x \gets x - a_i$。

现在考虑如何回答询问。对询问进行离线，**维护右端点 $r$ 确定时，每个左端点 $l$ 对应的答案**。注意到答案随着左端点的增大而增大。每当新加入一个新检查点，即 $r \gets r + 1$ 时。答案序列可以被分成三段（在线段树上二分找出），第一段不变，第二段区间赋值，第三段区间减。使用线段树维护即可。

时间复杂度 $\mathcal{O}((n + m)\log n)$。

```c++
#include <bits/stdc++.h>

#define debug(a) std::cout << #a << "=" << (a) << ' '

using s64 = long long;
using u64 = unsigned long long;

/* 取 min */
template <class T>
inline void chmin(T &x, const T &y) {
    if (x > y) {
        x = y;
    }
}
/* 取 max */
template <class T>
inline void chmax(T &x, const T &y) {
    if (x < y) {
        x = y;
    }
}

/* ----- ----- ----- 正文 ----- ----- ----- */

const int N = 300100, MaxQ = 300100;
const int inf = 0x3f3f3f3f;

int n, Q, D;
int a[N];

namespace SGT {
    struct node {
        int min;
        int tag; // 以 -inf 为空
        int add; // 以 0 为空
        void mk_tag(int x) {
            min = x;
            tag = x;
            add = 0;
        }
        void mk_add(int x) {
            min += x;
            add += x;
        }
    } t[N * 4];
    // 答案数组单调递增

    void upd(int p) {
        t[p].min = std::min(t[p * 2].min, t[p * 2 + 1].min);
    }

    void spread(int p) {
        if (t[p].tag != -inf) {
            t[p * 2].mk_tag(t[p].tag), t[p * 2 + 1].mk_tag(t[p].tag);
            t[p].tag = -inf;
        }
        if (t[p].add) {
            t[p * 2].mk_add(t[p].add), t[p * 2 + 1].mk_add(t[p].add);
            t[p].add = 0;
        }
    }

    void build(int p, int l, int r) {
        t[p].tag = -inf, t[p].add = 0;
        if (l == r) {
            t[p].min = D;
            return;
        }
        int mid = (l + r) >> 1;
        build(p * 2, l, mid), build(p * 2 + 1, mid + 1, r);
        upd(p);
    }

    void change_tag(int p, int l, int r, int s, int e, int x) {
        if (s <= l && r <= e) {
            t[p].mk_tag(x);
            return;
        }
        spread(p);
        int mid = (l + r) >> 1;
        if (s <= mid) {
            change_tag(p * 2, l, mid, s, e, x);
        }
        if (mid < e) {
            change_tag(p * 2 + 1, mid + 1, r, s, e, x);
        }
        upd(p);
    }

    void change_add(int p, int l, int r, int s, int e, int x) {
        if (s <= l && r <= e) {
            t[p].mk_add(x);
            return;
        }
        spread(p);
        int mid = (l + r) >> 1;
        if (s <= mid) {
            change_add(p * 2, l, mid, s, e, x);
        }
        if (mid < e) {
            change_add(p * 2 + 1, mid + 1, r, s, e, x);
        }
        upd(p);
    }

    int ask(int p, int l, int r, int x) {
        if (l == r) {
            return t[p].min;
        }
        spread(p);
        int mid = (l + r) >> 1;
        if (x <= mid) {
            return ask(p * 2, l, mid, x);
        } else {
            return ask(p * 2 + 1, mid + 1, r, x);
        }
    }

    int findLast(int p, int l, int r, int v) {
        if (v < t[p].min) {
            return l - 1;
        }
        if (l == r) {
            return l;
        }
        spread(p);
        int mid = (l + r) >> 1;
        if (t[p * 2 + 1].min <= v) {
            return findLast(p * 2 + 1, mid + 1, r, v);
        } else {
            return findLast(p * 2, l, mid, v);
        }
    }
}

std::vector< std::pair<int, int> > qry[MaxQ];
int ans[MaxQ];

int main() {
    std::ios::sync_with_stdio(0);
    std::cin.tie(0);

    std::cin >> n >> Q >> D;

    for (int i = 1; i <= n; i ++) {
        std::cin >> a[i];
    }

    SGT::build(1, 1, n);

    for (int i = 1; i <= Q; i ++) {
        int l, r;
        std::cin >> l >> r;

        qry[r].push_back({l, i});
    }

    for (int i = 1; i <= n; i ++) {
        int p1 = SGT::findLast(1, 1, n, a[i] - 1);
        int p2 = std::min(i, SGT::findLast(1, 1, n, 2 * a[i] - 1));

        // std::cout << ' ' << p1 << ' ' << p2 << '\n';

        if (p1 + 1 <= p2) {
            SGT::change_tag(1, 1, n, p1 + 1, p2, a[i] - 1);
        }
        if (p2 + 1 <= i) {
            SGT::change_add(1, 1, n, p2 + 1, i, -a[i]);
        }

        for (auto [l, id] : qry[i]) {
            ans[id] = SGT::ask(1, 1, n, l);
        }
    }

    for (int i = 1; i <= Q; i ++) {
        std::cout << ans[i] << '\n';
    }

    return 0;
}

/**
 *  心中无女人
 *  比赛自然神
 *  模板第一页
 *  忘掉心上人
**/

/*
3 1 10
7 6 2
1 3
*/
```