---
title: 「ABC390G」Permutation Concatenation
date: 2025-09-17 22:31:16
updated: 2025-09-17 22:31:16
categories: AtCoder
tags:
  - 「数学」
  - 多项式
  - 组合数学
---

# Description

Link：[ABC390G](https://atcoder.jp/contests/abc390/tasks/abc390_g)

{% note default %}

给出一个正整数 $n$。

对于一个长度为 $n$ 的整数序列 $A = \{a_1, a_2, \dots, a_n\}$，设 $f(A)$ 表示 $a_1 \sim a_n$ 顺次连接构成的十进制数值 $\overline{a_1\dots a_n}$。例如 $f(\{ 1, 20, 34 \}) = 12034$。

你需要对于 $n$ 的所有排列 $P$，求出 $f(P)$ 之和。答案对 $998244353$ 取模。

数据范围：$1 \leq n \leq 2 \times 10^5$。

时空限制：$2$s / $1024$MiB。

{% endnote %}

<!-- more -->

# Solution

对于一个整数序列 $a_1, a_2, \dots, a_n$，考虑 $a_i$ 对 $f(A)$ 的贡献，实际上即为 $a_i \times 10^{L}$（其中 $L$ 表示 $\overline{a_{i + 1}\dots a_n}$ 的位数）。

现在考虑计算一个数 $v$ 对整体答案的贡献。枚举其在排列中的位置 $i$，则此时要考虑 $i + 1 \sim n$ 部分的影响。等价于在 $\{1, \dots, n \} \setminus \{v\}$ 中选出 $n - i$ 个数，每当选到一个位数为 $d$ 的数，就会使得贡献乘上 $10^d$。求所有选法的贡献之和。

设 $d_j$ 表示数字 $j$ 的位数。会发现我们要求的即为
$$
[x^{n - i}] \left( \frac{\prod_{j = 1}^n (1 + 10^{d_j}x)}{(1 + 10^{d_v}x)} \right)
$$

容易发现，对于位数相同的数字 $v$，其需要求解的多项式是一样的。故我们可以**将位数相同的数字放在一起计算贡献**。

设 $\mathrm{cnt}_d, \mathrm{sum}_d$ 分别表示 $1 \sim n$ 中位数为 $d$ 的数字个数与数字之和。

**先考虑求多项式的分子**，发现 $d_j$ 相同的项仍然可以放在一起计算。位数为 $d$ 的数，对分子的贡献为 $\left( 1 + 10^d x \right)^{\mathrm{cnt}_d}$，可以用二项式定理快速得到该多项式。再将所有位数 $d$ 的贡献相乘即可得到分子。

有了分子以后，对于每个位数 $d$，**都可以用分子暴力除以 $(1 + 10^d x)$ 得到目标多项式**。记此时的目标多项式为 $\mathrm{cur}$，则所有位数为 $d$ 的数字对答案的贡献为

$$
\sum\limits_{i = 1}^n (i - 1)! \cdot (n - i)! \cdot \mathrm{sum}_d \cdot [x^{n - i}]\mathrm{cur}
$$

时间复杂度 $\mathcal{O}(n \log n)$。

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

struct BinomCoef {
    std::vector<int> fact, facv;

    void init(const int &n) {
        fact.resize(n + 1), facv.resize(n + 1);
        
        fact[0] = 1;
        for (int i = 1; i <= n; i ++) {
            fact[i] = 1ll * fact[i - 1] * i % mod;
        }

        facv[n] = qpow(fact[n], mod - 2, mod);
        for (int i = n - 1; i >= 0; i --) {
            facv[i] = 1ll * facv[i + 1] * (i + 1) % mod;
        }
    }

    int binom(int n, int m) {
        if (n < m || m < 0) {
            return 0;
        }
        return 1ll * facv[m] * facv[n - m] % mod * fact[n] % mod;
    }
} bc;

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

/*
    多项式全家桶（2025.09.17）
    C++23 以上：请放心使用
    C++11 ~ C++20：请删去所有的 constexpr（直接 #define constexpr 即可）
*/
struct poly : public std::vector<int> {
    poly() : std::vector<int>() {}
    explicit constexpr poly(int n) : std::vector<int>(n) {}
    explicit constexpr poly(const std::vector<int> &a) : std::vector<int>(a) {}
    constexpr poly(const std::initializer_list<int> &a) : std::vector<int>(a) {}

    template <class InputIt, class = std::_RequireInputIter<InputIt>>
    explicit constexpr poly(InputIt st, InputIt ed) : std::vector<int>(st, ed) {}

    // 多项式乘法
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

    // 多项式乘法（暴力）
    constexpr friend poly mul_bf(poly a, poly b) {
        poly c(a.size() + b.size() - 1);
        for (int i = 0; i < a.size(); i ++) {
            for (int j = 0; j < b.size(); j ++) {
                c[i + j] = (c[i + j] + 1ll * a[i] * b[j]) % mod;
            }
        }
        return c;
    }
    // 多项式除法（暴力）
    constexpr friend poly div_bf(poly a, poly b) {
        int m = b.size() - 1, inv = qpow(b[0], mod - 2, mod);
        poly c(a.size() - b.size() + 1);
        for (int i = 0; i < c.size(); i ++) {
            c[i] = a[i];
            for (int j = std::max(0, i - m); j < i; j ++) {
                dec(c[i], 1ll * c[j] * b[i - j] % mod);
            }
            mul(c[i], inv);
        }
        return c;
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

    constexpr poly mulT(poly b) const {
        if (b.size() == 0) return poly();
        std::reverse(b.begin(), b.end());
        return ((*this) * b).divxk(b.size() - 1);
        // 等价于“翻转卷积”，a.mulT(b) 相当于是求 a[i] * b[j] -> c[i - j]
    }

    // 多项式导数
    constexpr poly deriv() const {
        if (empty()) return poly();
        poly res(size() - 1);
        for (int i = 1; i < size(); i ++) {
            res[i - 1] = 1ll * (*this)[i] * i % mod;
        }
        return res;
    }

    // 多项式积分
    constexpr poly integ() const {
        poly res(size() + 1);
        for (int i = 0; i < size(); i ++) {
            res[i + 1] = 1ll * (*this)[i] * qpow(i + 1, mod - 2, mod) % mod;
        }
        return res;
    }

    // 多项式求逆
    constexpr poly inv(int m) const {
        poly x{qpow((*this)[0], mod - 2, mod)};
        int k = 1;
        while (k < m) {
            k <<= 1;
            x = (x * (poly{2} - modxk(k) * x)).modxk(k);
        }
        return x.modxk(m);
    }
    // 少项式求逆
    constexpr poly inv_bf(int m) const {
        int n = size() - 1, inv = qpow((*this)[0], mod - 2, mod);
        poly b(m);
        for (int i = 0; i < m; i ++) {
            b[i] = i == 0;
            for (int j = std::max(0, i - n); j < i; j ++) {
                dec(b[i], 1ll * b[j] * (*this)[i - j] % mod);
            }
            mul(b[i], inv);
        }
        return b;
    }

    // 多项式开方
    constexpr poly sqrt(int m) const {
        poly x{1}; // 默认常数项为 1
        int k = 1;
        while (k < m) {
            k <<= 1;
            x = (x + (modxk(k) * x.inv(k)).modxk(k)) / 2;
        }
        return x.modxk(m);
    }

    // 多项式带余除法
    constexpr friend std::pair<poly, poly> divmod(poly a, poly b) {
        int n = a.size(), m = b.size(), L = n - m + 1;
        if (n < m) return {poly{0}, a};
        poly q = (a.flip().modxk(L) * b.flip().inv(L)).modxk(L).flip();
        poly r = a - b * q; while (r.size() && !r.back()) r.pop_back();
        return {q, r};
    }

    // 多项式 ln
    constexpr poly ln(int m) const { // 需要保证常数项为 1
        return (deriv() * inv(m)).integ().modxk(m);
    }

    // 多项式 exp
    constexpr poly exp(int m) const { // 需要保证常数项为 0
        poly x{1};
        int k = 1;
        while (k < m) {
            k <<= 1;
            x = (x * (poly{1} - x.ln(k) + modxk(k))).modxk(k);
        }
        return x.modxk(m);
    }

    // 多项式快速幂
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
    // 少项式快速幂
    constexpr poly pow_bf(int k, int m) const {
        std::vector<int> v(m);
        v[1] = 1;
        for (int i = 2; i < m; i ++) {
            v[i] = 1ll * v[mod % i] * (mod - mod / i) % mod;
        }

        int n = size() - 1, inv = qpow((*this)[0], mod - 2, mod);
        poly b(m);
        b[0] = qpow((*this)[0], k, mod);
        for (int i = 0; i + 1 < m; i ++) {
            b[i + 1] = 0;
            for (int j = std::max(0, i - n + 1); j <= i; j ++) {
                add(b[i + 1], 1ll * (i - j + 1) * (*this)[i - j + 1] % mod * b[j] % mod);
            }
            mul(b[i + 1], k);
            for (int j = std::max(0, i - n); j < i; j ++) {
                dec(b[i + 1], 1ll * (j + 1) * b[j + 1] % mod * (*this)[i - j] % mod);
            }
            mul(b[i + 1], v[i + 1]), mul(b[i + 1], inv);
        }
        return b;
    }

    // 多项式多点求值
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

// 多项式快速插值
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

int n;

int digit(int x) {
    int cnt = 0;
    while (x) {
        x /= 10;
        cnt ++;
    }
    return cnt;
}

int maxd;
int sum[10], cnt[10];

int main() {
    std::ios::sync_with_stdio(0);
    std::cin.tie(0);

    std::cin >> n;

    bc.init(n);

    maxd = digit(n);
    for (int i = 1; i <= n; i ++) {
        int d = digit(i);
        add(sum[d], i), cnt[d] ++;
    }

    poly res{1};
    for (int d = 1, L = 10; d <= maxd; d ++, L *= 10) {
        poly a(cnt[d] + 1);
        for (int i = 0, w = 1; i <= cnt[d]; i ++, mul(w, L)) {
            a[i] = 1ll * bc.binom(cnt[d], i) * w % mod;
        }

        res *= a;
    }

    int ans = 0;
    for (int d = 1, L = 10; d <= maxd; d ++, L *= 10) {
        poly cur = div_bf(res, poly{1, L});
        for (int i = 1; i <= n; i ++) {
            add(ans, 1ll * bc.fact[i - 1] * bc.fact[n - i] % mod * sum[d] % mod * cur[n - i] % mod);
        }
    }

    std::cout << ans << '\n';

    return 0;
}

/**
 *  心中无女人
 *  比赛自然神
 *  模板第一页
 *  忘掉心上人
**/
```