---
title: 「Luogu P4389」付公主的背包
date: 2025-09-15 23:24:23
updated: 2025-09-15 23:24:23
categories: Luogu
tags:
  - 生成函数
  - 多项式
---

# Description

Link：[Luogu P4389](https://www.luogu.com.cn/problem/P4389)

{% note default %}

有 $n$ 种物品，其中第 $i$ 种物品的体积为 $v_i$，每一种物品的数量都是无限的。

给出背包的体积上限 $m$，对于 $s \in [1, m]$，你都需要求出使用这些物品恰好装满 $s$ 体积的方案数。两个方案不同当且仅当存在某个物品的选取个数不同。

数据范围：$1 \leq n, m \leq 10^5$，$1 \leq v_i \leq m$。

时空限制：$1$s / $500$MiB。

{% endnote %}

<!-- more -->

# Solution

{% note danger %}

有一个明显错误的做法：设 $c_i$ 表示体积为 $i$ 的物品个数，记 $c_i$ 的生成函数为 $a$，则答案的多项式为 $a + a^2 + a^3 + \cdots = \frac{a}{1 - a}$。

**错误原因是，方案被重复计算**。例如 $3$ 可以同时可以被 $1 + 2$ 和 $2 + 1$ 构造出来。

{% endnote %}

为了不重复计算，我们可以从一个物品的选取次数来入手。

对于一个体积为 $v$ 的物品，可以通过选取 $i(i \geq 0)$ 次获得一个体积为 $iv$ 的物品，故该物品对应的生成函数为 $1 + x^v + x^{2v} + \cdots = \frac{1}{1 - x^v}$。显然答案即为所有物品生成函数的乘积。

直接相乘不太好做。**此时可以考虑先取 ln，转化为相加后再进行 exp**。注意到该生成函数的 ln 形式相当特殊

$$
\ln \frac{1}{1 - x^v} = -\ln(1 - x^v) = \sum\limits_{i \geq 1} \frac{x^{vi}}{i}
$$

{% note default %}
<details>
<summary> 来回顾一下泰勒公式与麦克劳林公式 </summary>

泰勒公式：

$$
f(x) = \sum\limits_{i = 0}^{\infty} \frac{f^{(i)}(a)}{i!} (x - a)^i
$$

麦克劳林公式：

$$
f(x) = \sum\limits_{i = 0}^{\infty} \frac{f^{(i)}(0)}{i!} x^i
$$

常见函数的麦克劳林展开

$$
\begin{aligned}
e^x & = \sum\limits_{i = 0}^{\infty} \frac{x^i}{i!} \\
\ln(1 + x) & = \sum\limits_{i = 1}^{\infty} \frac{(-1)^{i - 1}x^i}{i} \\
\ln(1 - x) & = -\sum\limits_{i = 1}^{\infty} \frac{x^i}{i}
\end{aligned}
$$

</details>
{% endnote %}

于是可以考虑用一个桶，记录一下体积为 $v$ 的物品个数。然后一起计算贡献。得出多项式 ln 之和的时间复杂度与调和级数一致，为 $\mathcal{O}(m \log m)$。最后再 exp 回去即可。

时间复杂度 $\mathcal{O}(m \log m)$。

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



int main() {
    std::ios::sync_with_stdio(0);
    std::cin.tie(0);

    int n, m;
    std::cin >> n >> m;

    std::vector<int> cnt(m + 1);

    for (int i = 1; i <= n; i ++) {
        int v;
        std::cin >> v;
        cnt[v] ++;
    }

    std::vector<int> inv(m + 1);
    for (int i = 1; i <= m; i ++) {
        inv[i] = qpow(i, mod - 2, mod);
    }

    poly a(m + 1);
    for (int i = 1; i <= m; i ++) {
        for (int j = 1; j <= m / i; j ++) {
            add(a[i * j], 1ll * cnt[i] * inv[j] % mod);
        }
    }

    poly res = a.exp(m + 1);

    for (int i = 1; i <= m; i ++) {
        std::cout << res[i] << '\n';
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