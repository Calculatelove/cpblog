---
title: 「4th Ucup Stage 10」A. Automatized Mineral Classification
date: 2026-01-16 09:52:00
updated: 2026-01-16 09:52:00
categories: Ucup
tags:
  - 「交互题」
  - 随机化
  - 分治
---

# Description

Link：[QOJ 16000](https://qoj.ac/contest/2814/problem/16000)

{% note default %}

**This is an interactive problem.**

对于 $1, \dots, n$，有一个隐藏的序列 $a_i$，其中 $a_i$ 表示数字 $i$ 的颜色。

有一个队列（初始为空），可以进行以下两种操作：
- `+ id`：将元素 $\mathrm{id}$ 加入队列末尾。
- `-`：将队列开头的元素弹出。

每次操作之后，你都会得知当前队列中共有多少种不同的元素。

你的目标是，将所有元素按照颜色分组，依次给出每个组中的元素。

数据范围：$1 \leq n \leq 500$，操作至多进行 $35000$ 次。交互器不自适应。

时空限制：$1$s / $256$MiB。

{% endnote %}

<!-- more -->

# Solution

## 生日悖论：$\mathcal{O}(n\sqrt{n})$

将序列 $a$ 的元素依次加入队列，直到第一次出现重复的元素时，最后加入的元素 $x$ 一定与之前的某一个元素 $y$ 相同。然后我们清空队列，在清空队列的过程中我们就可以找到元素 $y$ 与 $x$。然后从序列 $a$ 中删除 $x$，重复上述的步骤。

直接这样暴力肯定有问题，不过我们可以将其打乱一下。在元素的顺序是随机的情况下，可以证明一次迭代的操作次数期望为 $\mathcal{O}(\sqrt{n})$。

这就是生日悖论，假设我们有 $n$ 个元素，分别来自 $k$ ($k \leq \frac{n}{2}$) 种不同的颜色。随机抽取 $\mathcal{O}(\sqrt{k})$ 个元素，就会出现重复的元素。该结论即使在颜色的分布是有偏的情况下，依然成立。

## 分治：$\mathcal{O}(n \log n)$

首先将序列 $a$ 的元素依次加入队列，每次当前颜色增加时，说明出现了一个新的颜色，将当前的元素视作该颜色的代表元素。这些代表元素构成集合 $R$。

设 $\mathrm{solve}(l, r, R)$ 表示求解区间 $[l, r]$ 中所有元素的颜色，其中区间 $[l, r]$ 所有颜色的并集构成 $R$。

假设往左半段递归，我们先将区间 $[l, \mathrm{mid}]$ 中所有元素加入队列，然后将集合 $R$ 中的元素加入队列，若 $r \in R$ 加入队列的时候当前颜色数量不变，则说明区间 $[l, \mathrm{mid}]$ 中存在颜色为 $r$ 的元素，需要将 $r$ 加入 $R_1$；否则说明区间 $[l, \mathrm{mid}]$ 不存在颜色为 $r$ 的元素。

然后我们将队列清空，并递归求解 $\mathrm{solve}(l, \mathrm{mid}, R_1)$。当然 $\mathrm{solve}(\mathrm{mid} + 1, r, R_2)$ 也是同理。

期望操作次数 $\mathcal{O}(n \log n)$。

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

const int N = 510;

int n, B;

int times;

int push(int x) {
    assert(++ times <= 35000);
    std::cout << "+ " << x << std::endl;
    int res;
    std::cin >> res;
    return res;
}
int pop() {
    assert(++ times <= 35000);
    std::cout << "-" << std::endl;
    int res;
    std::cin >> res;
    return res;
}

std::vector<int> key0;

std::vector<int> hold[N];

void solve(int l, int r, std::vector<int> key) {
    if (l == r) {
        assert(key.size() == 1);
        hold[key[0]].push_back(l);
        return;
    }
    int mid = (l + r) >> 1;

    std::vector<int> lkey, rkey;
    {
        int lst = 0;
        for (int x : key) {
            lst = push(x);
        }
        for (int i = l; i <= mid; i ++) {
            lst = push(i);
        }

        for (int x : key) {
            int net = pop();
            if (net == lst) {
                lkey.push_back(x);
            }
            lst = net;
        }
        for (int i = l; i <= mid; i ++) {
            pop();
        }
    }
    {
        int lst = 0;
        for (int x : key) {
            lst = push(x);
        }
        for (int i = mid + 1; i <= r; i ++) {
            lst = push(i);
        }

        for (int x : key) {
            int net = pop();
            if (net == lst) {
                rkey.push_back(x);
            }
            lst = net;
        }
        for (int i = mid + 1; i <= r; i ++) {
            pop();
        }
    }

    solve(l, mid, lkey);
    solve(mid + 1, r, rkey);
}

int main() {
    std::ios::sync_with_stdio(0);
    std::cin.tie(0);

    std::cin >> n;

    {
        int lst = 0;
        for (int i = 1; i <= n; i ++) {
            int net = push(i);
            if (net == lst + 1) {
                key0.push_back(i);
                lst = net;
            }
        }
        for (int i = 1; i <= n; i ++) {
            pop();
        }
    }

    solve(1, n, key0);

    int anslen = 0;
    for (int i = 1; i <= n; i ++) {
        if (hold[i].size()) {
            anslen ++;
        }
    }
    std::cout << "! " << anslen << std::endl;
    for (int i = 1; i <= n; i ++) {
        if (hold[i].size()) {
            std::cout << hold[i].size() << ' ';
            for (int x : hold[i]) {
                std::cout << x << ' ';
            }
            std::cout << std::endl;
        }
    }

    return 0;
}
```