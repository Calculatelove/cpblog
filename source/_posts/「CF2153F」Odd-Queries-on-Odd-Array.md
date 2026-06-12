---
title: 「CF2153F」Odd Queries on Odd Array
date: 2025-10-12 18:52:28
updated: 2025-10-12 18:52:28
categories: Codeforces
tags:
  - 「数据结构」
  - 树形结构
---

# Description

Link：[CF2153F](https://codeforces.com/contest/2153/problem/F)

{% note default %}

给出一个长度为 $n$ 的序列 $a_1, a_2, \dots, a_n$，满足不存在 $1 \leq i < j < k < l \leq n$ 使得 $a_i \neq a_j$，$a_i = a_k$，$a_j = a_l$。

有 $Q$ 次询问，每次询问给出两个整数 $l, r(1 \leq l \leq r \leq n)$，你需要求出区间 $[l, r]$ 中所有出现了奇数次的数之和。

**本题强制在线。**

数据范围：$1 \leq n, Q \leq 5\times 10^5$，$1 \leq a_i \leq n$，$1 \leq l \leq r \leq n$。

时空限制：$10$s / $1024$MiB。

{% endnote %}

<!-- more -->

记一种神秘性质，所构建的神奇的树形结构。

此题可以被分块通过（我们只关心出现次数的奇偶性，使用 `bool` 数组存储出现次数即可）。

# Solution

根据数组的神秘性质建树：
- 初始栈中只有一个节点 $0$。
- 依次遍历 $a_1, \dots, a_n$，假设当前遍历到了 $a_i$。
  - 将当前栈顶与节点 $i$ 连边，节点 $i$ 的点权为 $a_i$。
  - 若 $i$ 是 $a_i$ 的第一次出现位置，则将节点 $i$ 入栈。
  - 若 $i$ 是 $a_i$ 最后一次出现位置，则将栈顶节点出栈（可以证明出栈的节点点权恰为 $a_i$）。

{% note info %}

**性质 1**：每次出栈的节点，恰为 $a_i$ 第一次出现时入栈的节点。

因为从 $a_i$ 第一次出现到最后一次出现之间，内部的元素必定仅出现在该范围内（否则不满足给定条件），故内部的所有元素必定已出栈。

{% endnote %}

{% note info %}

**性质 2**：点权均为 $v$ 的节点，必定构成一个连通块。且第二次以后出现的节点无儿子，以第一次出现的节点为父亲。

证明：假设权值 $x$ 第一次出现时，加入了节点 $u$。另有一节点 $v$ 的权值等于 $x$，现在要证明 $u, v$ 直接相连。

假设 $u, v$ 之间存在一个节点 $z$ 的权值 $\neq x$，此时 $z$ 必定还未出栈，于是 $a_z$ 最后一次出现的位置必定在 $x$ 之后，就形成了 $\{x, a_z, x, a_z\}$ 的结构。与题目条件矛盾。

由于一种权值入栈的节点只有一个，于是第二次以后出现的节点无儿子，以第一次出现的节点为父亲。

{% endnote %}

{% note info %}

**性质 3**：该树的 dfs 序为 $0, 1, \dots, n$。

{% endnote %}

---

考虑如何回答询问。设 $u = \mathrm{LCA}(x, y)$，由于 dfs 序的性质，**区间 $[l, r]$ 可以拆成 $u$ 的若干个连续子树，以及 $l, r$ 所在的残缺子树**。注意特判一下 $u = l$ 的情况。

运用性质 2。以点 $u$ 分割 $u$ 的所有子树，对于 $u$ 每个儿子 $v$：
- 若 $a_v = a_u$，子树 $v$ 仅有 $v$ 一个孤立点。
- 若 $a_v \neq a_u$，**子树 $v$ 与外界相互独立**，（使用树上差分）提前预处理出子树 $v$ 的答案即可。

对于询问中的连续子树，我们使用前缀和预处理出 $a_v \neq a_u$ 部分的答案之和，同时也处理出 $a_v = a_u$ 的孤立点个数以便后续统计奇偶性。

对于 $l$ 所在的残缺子树：
- 若 $a_l = a_u$，则 $l$ 所在的残缺子树仅有一个孤立点 $l$，计入孤立点个数即可。
- 若 $a_l \neq a_u$，**则 $l$ 所在的残缺子树答案与外界独立**。设 $\mathrm{L}$ 表示 $l$ 所在子树的最大节点编号，我们要求的是 $a_{l, \dots, L}$ 的答案，使用 $\mathrm{suf}_l - \mathrm{suf}_{L + 1}$ 得到即可。

（$r$ 所在的残缺子树同理，需要预处理出每个前缀的答案）

最后统计一下有多少个权值为 $a_u$ 的孤立点，若为奇数个同样也要计入答案。

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

const int N = 500100;

int n, Q;
int a[N];

std::vector<int> pos[N];

std::vector<int> son[N];

void buildTree() {
    for (int i = 0; i <= n; i ++) {
        son[i].clear();
    }

    std::stack<int> s;
    s.push(0);

    for (int i = 1; i <= n; i ++) {
        int u = s.top();
        son[u].push_back(i);

        if (pos[a[i]].front() == i) {
            s.push(i);
        }
        if (pos[a[i]].back() == i) {
            s.pop();
        }
    }
}

s64 f[N];
s64 sum[N];

int lp[N], rp[N];

int dep[N];
int anc[20][N];

int lca(int x, int y) {
    if (dep[x] > dep[y]) {
        std::swap(x, y);
    }
    for (int i = 19; i >= 0; i --) {
        if (dep[x] <= dep[y] - (1 << i)) {
            y = anc[i][y];
        }
    }
    if (x == y) {
        return x;
    }
    for (int i = 19; i >= 0; i --) {
        if (anc[i][x] ^ anc[i][y]) {
            x = anc[i][x], y = anc[i][y];
        }
    }
    return anc[0][x];
}

void dfs_init(int u, int fu) {
    if (u) {
        dep[u] = dep[fu] + 1;
        anc[0][u] = fu;
        for (int i = 1; i <= 19; i ++) {
            anc[i][u] = anc[i - 1][anc[i - 1][u]];
        }
    }

    lp[u] = rp[u] = u;
    for (int v : son[u]) {
        dfs_init(v, u);
        sum[u] += sum[v];
        chmin(lp[u], lp[v]), chmax(rp[u], rp[v]);
    }
    f[u] += sum[u];
}

bool tmp[N];
s64 pre[N], suf[N];

std::vector<s64> s[N];
std::vector<int> c[N];

s64 ask(int l, int r) {
    if (l == r) {
        return a[l];
    }

    int u = lca(l, r);
    if (u == l) {
        int pr = std::upper_bound(son[u].begin(), son[u].end(), r) - son[u].begin() - 1;

        s64 ans = 0;

        int cnt = 1;
        if (pr) {
            ans += s[u][pr - 1];
            cnt += c[u][pr - 1];
        }

        if (a[r] == a[u]) {
            cnt ++;
        } else {
            ans += pre[r];
            ans -= pre[lp[son[u][pr]] - 1];
        }

        if (cnt & 1) {
            ans += a[u];
        }

        return ans;
    }

    int pl = std::upper_bound(son[u].begin(), son[u].end(), l) - son[u].begin() - 1,
        pr = std::upper_bound(son[u].begin(), son[u].end(), r) - son[u].begin() - 1;

    s64 ans = s[u][pr - 1] - s[u][pl];
    int cnt = c[u][pr - 1] - c[u][pl];

    if (a[l] == a[u]) {
        cnt ++;
    } else {
        ans += suf[l];
        ans -= suf[rp[son[u][pl]] + 1];
    }

    if (a[r] == a[u]) {
        cnt ++;
    } else {
        ans += pre[r];
        ans -= pre[lp[son[u][pr]] - 1];
    }

    if (cnt & 1) {
        ans += a[u];
    }

    return ans;
}

void work() {
    std::cin >> n >> Q;

    a[0] = 0;
    for (int i = 1; i <= n; i ++) {
        std::cin >> a[i];
    }

    for (int i = 0; i <= n; i ++) {
        pos[i].clear();
        f[i] = sum[i] = 0;
    }
    for (int i = 0; i <= n; i ++) {
        pos[a[i]].push_back(i);
    }

    buildTree();

    for (int v = 0; v <= n; v ++) {
        if (pos[v].size() == 0) {
            continue;
        }

        int p = pos[v][0];
        if (pos[v].size() & 1) {
            sum[p] += v;
        }

        for (int i = 1; i < pos[v].size(); i ++) {
            f[pos[v][i]] += v;
        }
    }

    dfs_init(0, 0);

    /* 前缀答案 */
    for (int i = 1; i <= n; i ++) {
        tmp[i] = 0;
    }

    pre[0] = 0;
    for (int i = 1; i <= n; i ++) {
        pre[i] = pre[i - 1];
        tmp[a[i]] ^= 1;
        if (tmp[a[i]]) {
            pre[i] += a[i];
        } else {
            pre[i] -= a[i];
        }
    }

    /* 后缀答案 */
    for (int i = 1; i <= n; i ++) {
        tmp[i] = 0;
    }

    suf[n + 1] = 0;
    for (int i = n; i >= 1; i --) {
        suf[i] = suf[i + 1];
        tmp[a[i]] ^= 1;
        if (tmp[a[i]]) {
            suf[i] += a[i];
        } else {
            suf[i] -= a[i];
        }
    }

    for (int u = 0; u <= n; u ++) {
        s[u].resize(son[u].size());
        c[u].resize(son[u].size());

        for (int i = 0; i < son[u].size(); i ++) {
            int v = son[u][i];

            if (a[v] == a[u]) {
                s[u][i] = 0;
                c[u][i] = 1;
            } else {
                s[u][i] = f[v];
                c[u][i] = 0;
            }

            if (i) {
                s[u][i] += s[u][i - 1];
                c[u][i] += c[u][i - 1];
            }
        }
    }

    s64 lastans = 0;
    while (Q --) {
        s64 l, r;
        std::cin >> l >> r;
 
        l = (l - 1 + lastans) % n + 1;
        r = (r - 1 + lastans) % n + 1;
        if (l > r) {
            std::swap(l, r);
        }
 
        std::cout << (lastans = ask(l, r)) << ' ';
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

/*
1
3 1
2 3 2
2 3
*/
```