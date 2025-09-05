---
title: 「2025 ICPC 武汉邀请赛」J. Dictionary
date: 2025-09-01 17:16:12
updated: 2025-09-01 17:16:12
categories: ICPC
tags:
  - SA
  - SAM
  - 线段树
---

# Description

Link：[QOJ 10745](https://qoj.ac/contest/2025/problem/10745)

{% note default %}

给出一个长度为 $n$ 的字符串 $S$。定义字典为 $S$ 的所有本质不同的子串构成的集合，定义单词为字典中的字符串。

有 $Q$ 次操作，每次操作会给出两个参数 $l, r$，你将会在字典中，学习以 $S[l : r]$ 为前缀的所有单词。每次操作结束后，你需要求出有多少个单词被你学习过。

数据范围：$1 \leq n, Q \leq 2 \times 10^5$，$1 \leq l \leq r \leq n$。

数据范围：$4$s / $1024$MiB。

{% endnote %}

<!-- more -->

唉西安邀请赛前，怎么这个题都不会做呢 ... 简直烂完了 ...

记录本题是因为，很少用 SA 的视角考虑这类问题，SA 相比 SAM 没那么直观。

# Solution

## 算法一：SA

考虑建出 SA。对于排名为 $i - 1$ 和 $i$ 的两个后缀 $\mathrm{sa}_{i - 1}, \mathrm{sa}_i$，其长度为 $1 \sim \mathrm{height}[i]$ 的前缀都是相同的，故后缀 $\mathrm{sa}_i$ 相比后缀 $\mathrm{sa}_{i - 1}$ 有 $n - \mathrm{sa}_i + 1 - \mathrm{height}[i]$ 个不同的前缀。记这些不同的前缀为**后缀 $\mathrm{sa}_i$ 表示的前缀**。

在任意时刻，每个后缀表示的前缀，被学习过的部分一定构成一个后缀。于是考虑维护没被学习过的部分构成的前缀大小。

每次操作 $l, r$，记 $u = \mathrm{rk}_l$。在 $u$ 两侧二分找到最大的区间 $[L, R]$，使得排名在该区间内的后缀与后缀 $l$ 的 LCP 均大于 $r - l + 1$。此时注意到**排名在 $(L, R]$ 内的后缀，其 height 值均比 $r - l + 1$ 大**，故这些后缀表示的前缀均要被学习（没学习过的部分全部清零）。特别地，**排名为 $L$ 的后缀，其 height 值可能小于等于 $r - l + 1$**，要学习的部分是一个后缀（没学习过的部分与特定值取 $\min$）。

使用线段树维护没被学习过的部分，支持区间赋值与单点修改。

时间复杂度 $\mathcal{O}((n + Q) \log n)$。

## 算法二：SAM

使用 SAM 做此题，就比 SA 直接得多了。对反串建 SAM，对于一个 SAM 上的状态表示的字符串集合，在任意时刻被学习过的部分一定构成一个后缀。于是考虑维护没被学习过的部分构成的前缀大小。

每次操作找到子串 $S[l : r]$ 在 SAM 上的状态 $u$（子串定位 trick）。在 parent 树上考虑，子树 $u$ 内的所有点（除了 $u$），表示的字符串集合均要被学习（没被学习过的部分全部清零）。状态 $u$ 表示的字符串集合要被学习的部分是一个后缀（没被学习过的部分与特定值取 $\min$）。

在 dfs 序上使用线段树维护没被学习过的部分，支持区间赋值与单点修改。

时间复杂度 $\mathcal{O}((n + Q) \log n)$。

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

const int N = 200100;
const int SIZE = N * 2;

int n, Q;
std::string str;

namespace SAM {
    int NodeCount = 1, Last = 1;
    struct node {
        int trans[26];
        int link, maxl;
    } t[SIZE];

    int create() {
        int p = ++ NodeCount;
        for (int i = 0; i < 26; i ++) t[p].trans[i] = 0;
        t[p].link = t[p].maxl = 0;
        return p;
    }

    void init() {
        NodeCount = 0;
        Last = create();
    }

    int location[N];

    std::vector<int> son[SIZE];
    int anc[19][SIZE];

    void extend(int c, int i) {
        int p = Last,
            np = Last = create();

        t[np].maxl = t[p].maxl + 1;

        for (; p && t[p].trans[c] == 0; p = t[p].link) t[p].trans[c] = np;

        if (!p) {
            t[np].link = 1;
        } else {
            int q = t[p].trans[c];
            if (t[q].maxl == t[p].maxl + 1) {
                t[np].link = q;
            } else {
                int nq = ++ NodeCount; t[nq] = t[q]; t[nq].maxl = t[p].maxl + 1;
                t[np].link = t[q].link = nq;
                for (; p && t[p].trans[c] == q; p = t[p].link) t[p].trans[c] = nq;
            }
        }

        location[i] = np;
    }

    void build_tree() {
        for (int i = 1; i <= NodeCount; i ++) {
            son[i].clear();
        }
        for (int i = 1; i <= NodeCount; i ++) {
            son[t[i].link].push_back(i);
            anc[0][i] = t[i].link;
        }
        for (int j = 1; j <= 18; j ++) {
            for (int i = 1; i <= NodeCount; i ++) {
                anc[j][i] = anc[j - 1][anc[j - 1][i]];
            }
        }
    }

    int find(int l, int r) {
        int p = location[l];
        for (int i = 18; i >= 0; i --) {
            if (t[anc[i][p]].maxl >= r - l + 1) {
                p = anc[i][p];
            }
        }
        return p;
    }
}

int dfsClock, dfn[SIZE], idx[SIZE];
int sze[SIZE];

void dfs_init(int u) {
    dfsClock ++;
    dfn[u] = dfsClock, idx[dfsClock] = u;
    sze[u] = 1;

    for (int v : SAM::son[u]) {
        dfs_init(v);
        sze[u] += sze[v];
    }
}

namespace SGT {
    struct node {
        s64 sum;
        int tag;
        void mk_tag() {
            sum = 0;
            tag = 1;
        }
    } t[SIZE * 4];

    void upd(int p) {
        t[p].sum = t[p * 2].sum + t[p * 2 + 1].sum;
    }

    void spread(int p) {
        if (t[p].tag) {
            t[p * 2].mk_tag(), t[p * 2 + 1].mk_tag();
            t[p].tag = 0;
        }
    }

    void build(int p, int l, int r) {
        t[p].tag = 0;
        if (l == r) {
            int u = idx[l];
            t[p].sum = SAM::t[u].maxl - SAM::t[SAM::t[u].link].maxl;
            return;
        }
        int mid = (l + r) >> 1;
        build(p * 2, l, mid), build(p * 2 + 1, mid + 1, r);
        upd(p);
    }

    void changePos(int p, int l, int r, int x, int y) {
        if (l == r) {
            t[p].sum = y;
            return;
        }
        spread(p);
        int mid = (l + r) >> 1;
        if (x <= mid) {
            changePos(p * 2, l, mid, x, y);
        } else {
            changePos(p * 2 + 1, mid + 1, r, x, y);
        }
        upd(p);
    }
    void changePos(int x, int y) {
        changePos(1, 1, dfsClock, x, y);
    }

    void changeRange(int p, int l, int r, int s, int e) {
        if (s <= l && r <= e) {
            t[p].mk_tag();
            return;
        }
        spread(p);
        int mid = (l + r) >> 1;
        if (s <= mid) {
            changeRange(p * 2, l, mid, s, e);
        }
        if (mid < e) {
            changeRange(p * 2 + 1, mid + 1, r, s, e);
        }
        upd(p);
    }
    void changeRange(int s, int e) {
        changeRange(1, 1, dfsClock, s, e);
    }

    s64 ask(int p, int l, int r, int s, int e) {
        if (s <= l && r <= e) {
            return t[p].sum;
        }
        spread(p);
        int mid = (l + r) >> 1;
        if (s <= mid && mid < e) {
            return ask(p * 2, l, mid, s, e) + ask(p * 2 + 1, mid + 1, r, s, e);
        }
        if (s <= mid) {
            return ask(p * 2, l, mid, s, e);
        } else {
            return ask(p * 2 + 1, mid + 1, r, s, e);
        }
    }
    s64 ask(int s, int e) {
        return ask(1, 1, dfsClock, s, e);
    }
}

s64 ans;

void op(int l, int r) {
    int p = SAM::find(l, r);

    if (sze[p] > 1) {
        ans += SGT::ask(dfn[p] + 1, dfn[p] + sze[p] - 1);
        SGT::changeRange(dfn[p] + 1, dfn[p] + sze[p] - 1);
    }

    int v = SGT::ask(dfn[p], dfn[p]), nv = r - l - SAM::t[SAM::t[p].link].maxl;
    if (v > nv) {
        ans += v - nv;
        SGT::changePos(dfn[p], nv);
    }
}

void work() {
    std::cin >> str, n = str.length();
    str = " " + str;

    SAM::init();

    for (int i = n; i >= 1; i --) {
        SAM::extend(str[i] - 'a', i);
    }

    SAM::build_tree();

    dfsClock = 0;
    dfs_init(1);

    ans = 0;
    SGT::build(1, 1, dfsClock);

    std::cin >> Q;

    while (Q --) {
        int l, r;
        std::cin >> l >> r;

        op(l, r);
        
        std::cout << ans << ' ';
    }

    std::cout << '\n';
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