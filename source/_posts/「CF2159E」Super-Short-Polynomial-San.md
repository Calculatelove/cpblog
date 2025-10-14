---
title: 「CF2159E」Super-Short-Polynomial-San
date: 2025-10-14 14:34:09
updated: 2025-10-14 14:34:09
categories: Codeforces
tags:
  - 多项式
---

# Description

Link：[CF2159E](https://codeforces.com/contest/2159/problem/E)

{% note default %}

已知三个整数 $a, b, c$。定义 $F_n(x) = (ax^2 + bx + c)^n$。

有 $Q$ 次询问，每次询问给出两个参数 $n, k$，你需要求出 $\sum_{i = 0}^k[x^i]F_n(x)$。答案对 $10^9 + 7$ 取模。

**本题强制在线**。

数据范围：$1 \leq a, b, c \leq 10^9 + 6$，$1 \leq Q \leq 3\times 10^5$，$0 \leq n \leq 3\times 10^5$，$0 \leq k \leq 2n$。

时空限制：$7$s / $1024$MiB。

{% endnote %}

<!-- more -->

# Solution

先考虑每次询问 $[x^k]F_n(x)$ 要如何回答。

考虑分块，记块长 $B$。
- 对于 $0 \leq i \leq \frac{n}{B}$，使用 "少项式快速幂" 的技巧预处理出 $\mathcal{F}_i(x) = F_{iB}(x)$。
- 对于 $0 \leq i < B$，暴力预处理出 $F_i(x)$。

则 $F_n(x)$ 可以拆成 $\mathcal{F}_{\lfloor n / B \rfloor}(x) \times F_{n \bmod B}(x)$。

注意到提取多项式 $P\times Q$ 的某一项系数，只需要 $\mathcal{O}(\min(\deg P, \deg Q))$ 的开销。于是每次询问只需 $\mathcal{O}(B)$ 的开销。

时间复杂度 $\mathcal{O}(\frac{n^2}{B} + B^2 + QB)$，取 $B = 1000$ 是一个比较好的选择。

现在考虑如何将询问的内容转化为前缀和。

设 $G_n(x)$ 满足 $[x^k]G_n(x) = \sum_{i = 0}^{k}[x^i]F_n(x)$，也就是 $G_n(x)$ 的系数是 $F_n(x)$ 系数的前缀和。

从计数的角度，可以得到性质 $G_{n + 1}(x) = G_n(x) \times (ax^2 + bx + c)$，进一步可以得到 $G_{n + m}(x) = G_n(x) \times F_m(x)$。**于是 $G_n(x)$ 同样可以拆成 $\mathcal{G}_{\lfloor n / B \rfloor}(x) \times F_{n \bmod B}(x)$**。

求出 $\mathcal{F}$ 之后对其做一遍前缀和即可得到 $\mathcal{G}$。

时间复杂度同上。

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

const int mod = 1e9 + 7; // 模数需根据实际问题调整

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
/* 模意义下 取反 */
inline void neg(int &x) {
    if (x) {
        x = mod - x;
    }
}
/* 模意义下 乘法 */
inline void mul(int &x, const int &y) {
    x = 1ll * x * y % mod;
}

/* 模意义下 修正 */
inline int norm(int x) {
    x %= mod;
    return x < 0 ? x + mod : x;
}

/* 快速幂 */
constexpr int qpow(int a, int b, int p) {
    int ans = 1;
    for (; b; b >>= 1) {
        if (b & 1) ans = 1ll * ans * a % p;
        a = 1ll * a * a % p;
    }
    return ans;
}

/**
 *    多项式全家桶 使用手册
 *    author：Calculatelove
 *    version：2025.10.14
 * 
 *  版本兼容：
 *  - C++23 以上：请放心使用
 *  - C++17 ~ C++20：
 *      请删去所有 poly 中的 constexpr（在 poly 上方 #define constexpr 即可）
 *  - C++14：
 *      请将 RootPower 前面的 constexpr 改成 const
 *  - C++11：
 *      请删去所有函数体前面的 constexpr，
 *      请将 findRootPower 与 RootPower 前面的 auto 改成 std::array<int, 32>
 * 
 *  模数：
 *  - 涉及到多项式乘法时：
 *      必须为 NTT 模数！否则请将 "多项式乘法" 替换为 "MTT 版本的多项式乘法"
 *  - 仅涉及到 poly 的四则运算、求导、积分，以及暴力多项式 mul_bf, div_bf, inv_bf，pow_bf 时：
 *      只需模数为质数即可
 * 
 *  原根：
 *  - 默认为 3，请根据具体 NTT 模数确认原根（在 findRootPower() 中修改）
 * 
 *  已有的常数优化：
 *  - 使用了一些函数辅助取模（0x11 取模.cpp）
 *  - 使用 findRootPower() 预处理原根的幂（即单位根）
 * 
 *  可能的常数优化：
 *  - 半在线卷积中，补充界只需 >= r - l + 1
 *  - 预处理阶乘以及阶乘逆元
 *      polyinit_factorial() -> shift(), shift_conti()
 *  - 预处理线性逆元
 *      polyinit_linearInverse() -> integ(), pow_bf()
 */
 
std::vector<int> v;
void polyinit_linearInverse(const int &n) {
    v.resize(n + 1);
    v[1] = 1;
    for (int i = 2; i <= n; i ++) {
        v[i] = 1ll * v[mod % i] * (mod - mod / i) % mod;
    }
}

// 预处理单位根
constexpr auto findRootPower() {
    int g = 3; // 模数对应的原根，需根据实际问题调整
    std::array<int, 32> w{};
    for (int k = 1, idx = 0; (mod - 1) % (k << 1) == 0; k <<= 1) {
        w[idx ++] = qpow(g, (mod - 1) / (k << 1), mod); // 2k 阶单位根
    }
    return w;
}

constexpr auto RootPower = findRootPower();

std::vector<int> rev;
void dft(std::vector<int> &a) {
    int n = a.size();
    for (int i = 0; i < n; i ++) {
        if (i < rev[i]) {
            std::swap(a[i], a[rev[i]]);
        }
    }
    for (int k = 1, idx = 0; k < n; k <<= 1) {
        int omega = RootPower[idx ++];
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

/* 多项式全家桶 */
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

        int L = 1, P = 0;
        while (L < tot) L <<= 1, P ++;
        rev.resize(L);
        for (int i = 0; i < L; i ++) {
            rev[i] = (rev[i >> 1] >> 1) | ((i & 1) << (P - 1));
        }

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
            neg(x);
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

    constexpr poly circ(int k) const {
        auto res = *this;
        for (int i = k; i < size(); i ++) {
            add(res[i - k], res[i]);
        }
        return res.modxk(k);
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
    constexpr poly inv(int m) const { // 需要保证常数项不为 0
        poly x{qpow((*this)[0], mod - 2, mod)};
        int k = 1;
        while (k < m) {
            k <<= 1;
            x = (x * (poly{2} - modxk(k) * x)).modxk(k);
        }
        return x.modxk(m);
    }
    // 少项式求逆
    constexpr poly inv_bf(int m) const { // 需要保证常数项不为 0
        auto &&a = *this;
        int n = size() - 1, inv = qpow(a[0], mod - 2, mod);

        poly b(m);
        for (int i = 0; i < m; i ++) {
            b[i] = i == 0;
            for (int j = std::max(0, i - n); j < i; j ++) {
                dec(b[i], 1ll * b[j] * a[i - j] % mod);
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
        auto &&a = *this;
        int n = size() - 1, inv = qpow(a[0], mod - 2, mod);

        poly b(m);
        b[0] = qpow(a[0], k, mod);
        for (int i = 0; i + 1 < m; i ++) {
            b[i + 1] = 0;
            for (int j = std::max(0, i - n + 1); j <= i; j ++) {
                add(b[i + 1], 1ll * (i - j + 1) * a[i - j + 1] % mod * b[j] % mod);
            }
            mul(b[i + 1], k);
            for (int j = std::max(0, i - n); j < i; j ++) {
                dec(b[i + 1], 1ll * (j + 1) * b[j + 1] % mod * a[i - j] % mod);
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
                t[p] = poly{1, norm(-x[l])};
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

    // 多项式平移
    constexpr poly shift(int c) const {
        int n = size() - 1;

        std::vector<int> fact(n + 1), facv(n + 1);
        fact[0] = 1;
        for (int i = 1; i <= n; i ++) {
            fact[i] = 1ll * fact[i - 1] * i % mod;
        }
        facv[n] = qpow(fact[n], mod - 2, mod);
        for (int i = n - 1; i >= 0; i --) {
            facv[i] = 1ll * facv[i + 1] * (i + 1) % mod;
        }

        c = norm(c);
        poly a(n + 1), b(n + 1);
        for (int i = 0; i <= n; i ++) {
            a[i] = 1ll * (*this)[i] * fact[i] % mod;
        }
        for (int i = 0, w = 1; i <= n; i ++, mul(w, c)) {
            b[i] = 1ll * w * facv[i] % mod;
        }

        poly res = a.mulT(b);
        for (int i = 0; i <= n; i ++) {
            mul(res[i], facv[i]);
        }

        return res;
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
            t[p] = poly{norm(-x[l]), 1};
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

// 连续点值平移
constexpr std::vector<int> shift_conti(std::vector<int> y, int c) {
    int n = y.size() - 1; // y 的下标从 0 开始

    c = norm(c);
    if (c == 0) return y;
    if (c <= n) {
        auto cur = std::vector<int>(y.begin() + c, y.end());
        auto net = shift_conti(y, n + 1);
        for (int i = 0; i < c; i ++) cur.push_back(net[i]);
        return cur;
    }

    std::vector<int> fact(n + 1), facv(n + 1);
    fact[0] = 1;
    for (int i = 1; i <= n; i ++) {
        fact[i] = 1ll * fact[i - 1] * i % mod;
    }
    facv[n] = qpow(fact[n], mod - 2, mod);
    for (int i = n - 1; i >= 0; i --) {
        facv[i] = 1ll * facv[i + 1] * (i + 1) % mod;
    }

    poly a(2 * n + 1), b(n + 1);
    for (int i = 0; i <= 2 * n; i ++) {
        a[i] = qpow(c - n + i, mod - 2, mod);
    }
    for (int i = 0; i <= n; i ++) {
        b[i] = 1ll * facv[i] * facv[n - i] % mod * y[i] % mod;
        if ((n - i) & 1) neg(b[i]);
    }

    poly res = a * b;

    int num = 1;
    for (int i = c - n; i <= c; i ++) mul(num, i);

    std::vector<int> ans(n + 1);
    for (int i = 0; i <= n; i ++) {
        ans[i] = 1ll * num * res[i + n] % mod;
        mul(num, c + i + 1), mul(num, qpow(c + i - n, mod - 2, mod));
    }
    return ans;
}

const int m = 3e5;
const int B = 1000;

int a, b, c;

int main() {
    std::ios::sync_with_stdio(0);
    std::cin.tie(0);

    std::cin >> a >> b >> c;
    poly f{c, b, a};

    polyinit_linearInverse(m * 2);

    std::vector<poly> g1(m / B + 1);

    g1[0] = {1};
    for (int i = 1; i < g1.size(); i ++) {
        g1[i] = f.pow_bf(i * B, i * B * 2 + 1);
        for (int j = 1; j < g1[i].size(); j ++) {
            add(g1[i][j], g1[i][j - 1]);
        }
    }

    std::vector<poly> f2(B + 1);

    f2[0] = {1};
    for (int i = 1; i < f2.size(); i ++) {
        f2[i] = mul_bf(f2[i - 1], f);
    }

    auto ask = [&] (int n, int k) {
        int p = n / B, q = n % B;

        int ans = 0;
        for (int i = 0; i < f2[q].size(); i ++) {
            int j = k - i;
            if (j >= (int)g1[p].size()) j = (int)g1[p].size() - 1;

            if (j >= 0) {
                add(ans, 1ll * f2[q][i] * g1[p][j] % mod);
                // debug(i),
                // debug(j),
                // debug(f2[q][i]), debug(g1[p][j]) << '\n';
            }
        }

        return ans;
    };

    int Q;
    std::cin >> Q;

    int lastans = 0;
    while (Q --) {
        int n, k;
        std::cin >> n >> k;
        n ^= lastans, k ^= lastans;

        std::cout << (lastans = ask(n, k)) << '\n';
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