---
title: 「2024 ICPC 昆明站」I. Items
date: 2025-09-13 11:29:14
updated: 2025-09-13 11:29:14
categories: ICPC
tags:
  - 「数学」
  - 生成函数
  - 多项式
---

# Description

Link：[QOJ 9870](https://qoj.ac/contest/1871/problem/9870)

{% note default %}

有 $n$ 种物品，其中第 $i$ 种物品的重量为 $w_i$，每一种物品的数量都是无限的。

是否能恰好选 $n$ 个物品，使得所选物品的重量之和为 $m$。

数据范围：$1 \leq n \leq 10^5$，$0 \leq m \leq n^2$，$0 \leq w_i \leq n$。

时空限制：$8$s / $1024$MiB。

{% endnote %}

<!-- more -->

# Solution

设 $b_i$ 表示是否存在重量为 $i$ 的物品，设 $f(x)$ 是 $b_i$ 的生成函数，则相当于是要判断 $[x^m]f^n(x)$ 是否大于 $0$。直接计算 $f^n(x)$ 肯定不行，因为值域太大了。

考虑进行一些转换，记 $B = \left\lfloor \frac{m}{n} \right\rfloor$，则令 $w_i \gets w_i - B$，$m \gets m - nB$。显然转化后的问题与原问题等价，此时 $w_i \in [-B, n - B]$，$m \in [0, n)$。

{% note info %}

结论：假设存在一种选择物品的方案，使得重量之和恰好为 $m$。则一定可以对选择物品的顺序进行重排序，使得任意时刻下（排序后的任何前缀），物品重量之和都在 $[0, n]$ 范围内。

证明：设剩余可供选择物品的可重集为 $S$。我们依次确定每次选择的物品是什么，设 $\mathrm{sum}$ 表示当前选择的物品的重量之和：
- 若 $\mathrm{sum} = B$，由于物品的重量范围为 $[-B, n - B]$，从 $S$ 中任意选择一个物品，都可以保证重量之和在 $[0, n]$ 范围内。
- 若 $\mathrm{sum} < B$，由于物品重量 $\leq n - B$，从 $S$ 中任意选择一个正的物品，都可以保证重量之和 $\leq n$。
  特别地，若 $S$ 中没有正的物品，则剩下的物品全是负的并且总和恰好为 $m - \mathrm{pre}$，以任意顺序加入剩下的物品即可。
- 若 $\mathrm{sum} > B$，由于物品重量 $\geq -B$，从 $S$ 中任意选择一个负的物品，都可以保证重量之和 $\geq 0$。
  特别地，若 $S$ 中没有负的物品，则剩下的物品全是正的并且总和恰好为 $m - \mathrm{pre}$，以任意顺序加入剩下的物品即可。

{% endnote %}

于是，**任意时刻下，有效的物品重量之和都在 $[0, n]$ 范围内**。于是我们沿用多项式快速幂的做法，每次只需保留 $x^{-n} \sim x^{n}$ 中的项即可。

理论上需要双模 NTT 来减少冲突的概率，但我的模板还没有这项功能，待填坑。

时间复杂度 $\mathcal{O}(n \log^2 n)$。

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

const int mod = 998244353; // 模数需根据实际问题调整

/* 模意义下 加法 */
inline void add(int &x, const int &y) {
    x += y;
    if (x >= mod) {
        x -= mod;
    }
}

/* 模意义下 减法 */
inline void dec(int &x, const int &y) {
    x -= y;
    if (x < 0) {
        x += mod;
    }
}

/* 模意义下 乘法 */
inline void mul(int &x, const int &y) {
    x = 1ll * x * y % mod;
}

/* 快速幂 */
int qpow(int a, int b, int p) {
    int ans = 1;
    for (; b; b >>= 1) {
        if (b & 1) ans = 1ll * ans * a % p;
        a = 1ll * a * a % p;
    }
    return ans;
}

int L, P;
std::vector<int> rev;

void poly_init(const int &n) {
    L = 1, P = 0;
    while (L < n) L <<= 1, P ++;
    rev.resize(L);
    for (int i = 1; i < L; i ++) {
        rev[i] = (rev[i >> 1] >> 1) | ((i & 1) << (P - 1));
    }
}

void dft(std::vector<int> &a) {
    int n = a.size();
    for (int i = 0; i < n; i ++) {
        if (i < rev[i]) {
            std::swap(a[i], a[rev[i]]);
        }
    }
    int g = 3;
    for (int k = 1; k < n; k <<= 1) {
        int omega = qpow(g, (mod - 1) / (k << 1), mod); // 2k 阶单位根
        for (int i = 0; i < n; i += (k << 1)) {
            int x = 1;
            for (int j = 0; j < k; j ++, mul(x, omega)) {
                int u = a[i + j], v = 1ll * x * a[i + j + k] % mod;
                add(a[i + j] = u, v), dec(a[i + j + k] = u, v);
            }
        }
    }
}

void idft(std::vector<int> &a) {
    int n = a.size(), inv = qpow(n, mod - 2, mod);
    std::reverse(a.begin() + 1, a.end());
    dft(a);
    for (int i = 0; i < n; i ++) {
        mul(a[i], inv);
    }
}

struct poly : public std::vector<int> {
    poly() : std::vector<int>() {}
    explicit constexpr poly(int n) : std::vector<int>(n) {}
    explicit constexpr poly(const std::vector<int> &a) : std::vector<int>(a) {}
    constexpr poly(const std::initializer_list<int> &a) : std::vector<int>(a) {}

    template <std::input_iterator InputIt>
    explicit constexpr poly(InputIt st, InputIt ed) : std::vector<int>(st, ed) {}

    constexpr friend poly operator * (poly a, poly b) {
        int tot = a.size() + b.size() - 1;
        if (tot < 128) {
            poly c(tot);
            for (int i = 0; i < a.size(); i ++) {
                for (int j = 0; j < b.size(); j ++) {
                    c[i + j] = (c[i + j] + 1ll * a[i] * b[j]) % mod;
                }
            }
            return c;
        }
        poly_init(tot);
        a.resize(L), b.resize(L);
        dft(a), dft(b);
        for (int i = 0; i < L; i ++) {
            mul(a[i], b[i]);
        }
        idft(a);
        a.resize(tot);
        return a;
    }

    constexpr friend poly operator + (poly a, poly b) {
        poly c(std::max(a.size(), b.size()));
        for (int i = 0; i < a.size(); i ++) {
            add(c[i], a[i]);
        }
        for (int i = 0; i < b.size(); i ++) {
            add(c[i], b[i]);
        }
        return c;
    }
    constexpr friend poly operator - (poly a, poly b) {
        poly c(std::max(a.size(), b.size()));
        for (int i = 0; i < a.size(); i ++) {
            add(c[i], a[i]);
        }
        for (int i = 0; i < b.size(); i ++) {
            dec(c[i], b[i]);
        }
        return c;
    }
    constexpr friend poly operator - (poly a) {
        for (int &x : a) {
            x = x ? mod - x : 0;
        }
        return a;
    }

    constexpr friend poly operator * (poly a, int b) {
        for (int &x : a) {
            mul(x, b);
        }
        return a;
    }
    constexpr friend poly operator / (poly a, int b) {
        int inv = qpow(b, mod - 2, mod);
        for (int &x : a) {
            mul(x, inv);
        }
        return a;
    }

    constexpr poly &operator += (poly b) {
        return (*this) = (*this) + b;
    }
    constexpr poly &operator -= (poly b) {
        return (*this) = (*this) - b;
    }
    constexpr poly &operator *= (poly b) {
        return (*this) = (*this) * b;
    }
    constexpr poly &operator *= (int b) {
        return (*this) = (*this) * b;
    }
    constexpr poly &operator /= (int b) {
        return (*this) = (*this) / b;
    }

    constexpr poly mulxk(int k) const {
        auto res = *this;
        res.insert(res.begin(), k, 0);
        return res;
    }
    constexpr poly divxk(int k) const {
        if (k >= size()) return poly();
        return poly((*this).begin() + k, (*this).end());
    }
    constexpr poly modxk(int k) const {
        if (k >= size()) return *this;
        return poly((*this).begin(), (*this).begin() + k);
    }

    constexpr poly flip() const {
        auto res = *this;
        std::reverse(res.begin(), res.end());
        return res;
    }

    constexpr poly deriv() const {
        if (empty()) return poly();
        poly res(size() - 1);
        for (int i = 1; i < size(); i ++) {
            res[i - 1] = 1ll * (*this)[i] * i % mod;
        }
        return res;
    }

    constexpr poly integ() const {
        poly res(size() + 1);
        for (int i = 0; i < size(); i ++) {
            res[i + 1] = 1ll * (*this)[i] * qpow(i + 1, mod - 2, mod) % mod;
        }
        return res;
    }

    constexpr poly inv(int m) const {
        poly x{qpow((*this)[0], mod - 2, mod)};
        int k = 1;
        while (k < m) {
            k <<= 1;
            x = (x * (poly{2} - modxk(k) * x)).modxk(k);
        }
        return x.modxk(m);
    }

    constexpr poly sqrt(int m) const {
        poly x{1}; // 默认常数项为 1
        int k = 1;
        while (k < m) {
            k <<= 1;
            x = (x + (modxk(k) * x.inv(k)).modxk(k)) / 2;
        }
        return x.modxk(m);
    }

    constexpr friend std::pair<poly, poly> div(poly a, poly b) {
        int n = a.size(), m = b.size(), L = n - m + 1;
        if (n < m) return {poly{0}, a};
        poly q = (a.flip().modxk(L) * b.flip().inv(L)).modxk(L).flip();
        poly r = a - b * q; while (r.size() && !r.back()) r.pop_back();
        return {q, r};
    }

    constexpr poly ln(int m) const { // 需要保证常数项为 1
        return (deriv() * inv(m)).integ().modxk(m);
    }

    constexpr poly exp(int m) const { // 需要保证常数项为 0
        poly x{1};
        int k = 1;
        while (k < m) {
            k <<= 1;
            x = (x * (poly{1} - x.ln(k) + modxk(k))).modxk(k);
        }
        return x.modxk(m);
    }

    constexpr poly pow(int k, int m) const {
        int i = 0;
        while (i < size() && (*this)[i] == 0) {
            i ++;
        }
        if (i == size() || 1ll * i * k >= m) {
            return poly(m);
        }
        int v = (*this)[i];
        poly f = divxk(i) * qpow(v, mod - 2, mod);
        return (f.ln(m - i * k) * k).exp(m - i * k).mulxk(i * k) * qpow(v, k, mod);
        /*
            当实际 k < mod 时，可以放心使用
            当实际 k >= mod 时，注意：
            - 当 i > 0 时（即常数项为 0），直接返回长度为 m 的全 0 多项式
            - 当 i = 0 时
              - 在多项式 ln 与多项式 exp 部分，k 需要对 mod 取模
              - 在 qpow(v, k, mod) 部分，k 需要对 mod - 1 取模
        */
    }

    constexpr poly mulT(poly b) const {
        if (b.size() == 0) return poly();
        std::reverse(b.begin(), b.end());
        return ((*this) * b).divxk(b.size() - 1);
    }
    constexpr std::vector<int> eval(std::vector<int> x) const {
        if (size() == 0) {
            return std::vector<int>(x.size(), 0);
        }

        int n = std::max(x.size(), this->size());
        std::vector<poly> t(n * 4);
        std::vector<int> ans(x.size());
        x.resize(n);

        std::function<void(int, int, int)> build = [&] (int p, int l, int r) {
            if (l == r) {
                int v = x[l];
                t[p] = poly{1, v ? mod - v : 0};
                return;
            }
            int mid = (l + r) >> 1;
            build(p * 2, l, mid), build(p * 2 + 1, mid + 1, r);
            t[p] = t[p * 2] * t[p * 2 + 1];
        };
        build(1, 0, n - 1);

        std::function<void(int, int, int, const poly &)> solve = [&] (int p, int l, int r, const poly &num) {
            if (l >= ans.size()) return;
            if (l == r) {
                ans[l] = num[0];
                return;
            }
            int mid = (l + r) >> 1;
            solve(p * 2, l, mid, num.mulT(t[p * 2 + 1]).modxk(mid - l + 1));
            solve(p * 2 + 1, mid + 1, r, num.mulT(t[p * 2]).modxk(r - mid));
        };
        solve(1, 0, n - 1, mulT(t[1].inv(n)));

        return ans;
    }
};

constexpr poly lagrange(std::vector< std::pair<int, int> > seq) {
    int n = seq.size();
    std::vector<poly> t(n * 4);
    std::vector<int> x(n);
    for (int i = 0; i < n; i ++) {
        x[i] = seq[i].first;
    }

    std::function<void(int, int, int)> build = [&] (int p, int l, int r) {
        if (l == r) {
            int v = x[l];
            t[p] = poly{v ? mod - v : 0, 1};
            return;
        }
        int mid = (l + r) >> 1;
        build(p * 2, l, mid), build(p * 2 + 1, mid + 1, r);
        t[p] = t[p * 2] * t[p * 2 + 1];
    };
    build(1, 0, n - 1);

    auto res = (t[1].deriv()).eval(x);
    std::function<poly(int, int, int)> solve = [&] (int p, int l, int r) {
        if (l == r) {
            int v = 1ll * seq[l].second * qpow(res[l], mod - 2, mod) % mod;
            return poly{v};
        }
        int mid = (l + r) >> 1;
        return solve(p * 2, l, mid) * t[p * 2 + 1] + solve(p * 2 + 1, mid + 1, r) * t[p * 2];
    };
    return solve(1, 0, n - 1);
}

int n; s64 m;
int b;

void work() {
    std::cin >> n >> m;

    b = m / n, m = m - 1ll * b * n;

    poly a(2 * n + 1);
    for (int i = 1; i <= n; i ++) {
        int w;
        std::cin >> w;
        w = w - b + n;
        a[w] = 1;
    }

    poly f = a;
    for (int b = n - 1; b; b >>= 1) {
        if (b & 1) f = (f * a).divxk(n).modxk(2 * n + 1);
        a = (a * a).divxk(n).modxk(2 * n + 1);
    }

    if (f[m + n]) {
        std::cout << "Yes\n";
    } else {
        std::cout << "No\n";
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