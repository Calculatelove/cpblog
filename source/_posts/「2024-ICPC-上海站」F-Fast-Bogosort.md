---
title: 「2024 ICPC 上海站」F. Fast Bogosort
date: 2025-10-10 10:51:27
updated: 2025-10-10 10:51:27
categories:
tags:
---

# Description

Link：[QOJ 9042](https://qoj.ac/contest/1913/problem/9042?v=1)

{% note default %}

给出一个关于 $n$ 的排列 $a$，对排列 $a$ 使用 Fast Bogosort：
- 如果数组已经有序，则算法终止。
- 否则，将数组划分成尽可能多的区间，每个区间 $[l, r]$ 恰好包含 $[l, r]$ 中的所有数字。注意区间不能重叠，并且他们的并集为 $[1, n]$。
- 对于每个被划分的区间 $[l, r]$，如果 $l < r$，则调用 `shuffle(l, r)`，在区间 $[l, r]$ 内均匀随机打乱数字。

现在，请你计算对排列 $a$ 使用 Fast Bogosort，`shuffle` 函数的期望调用次数。

数据范围：$1 \leq n \leq 10^5$。

时空限制：$3$s / $1024$MiB。

{% endnote %}

<!-- more -->

做此类推式子题，可以先写暴力验证。

# Solution

注意到，数组划分出来的区间，只会继续分裂成小区间。也就是说，每次操作后，当前问题都有可能变成规模更小的子问题。

设 $g_i$ 表示有多少个长度为 $i$ 的排列，只能恰好分出一段。则有

$$
g_i = i! - \sum_{j = 1}^{i - 1} g_j \cdot (i - j)!
$$

设 $f_i$ 表示长度为 $i$ 的段，使用 Fast Bogosort 后 `shuffle` 函数的期望调用次数（第一次 `shuffle` 暂且不算）。枚举第一个分段 $j$，则有

$$
f_i = \sum_{j = 1}^i \frac{(i - j)!g_j}{i!} (f_j + [j > 1] + f_{i - j})
$$

（注意：这里每个分段第一次 `shuffle` 的贡献，在首次枚举到的 $j$ 之中计算）

进一步移项，得

$$
f_i = \frac{1}{i! - g_i} \left( \sum_{j = i}^{i - 1} (i - j)g_j (f_j + [j > 1] + f_{i - j}) \right) + \frac{g_i}{i! - g_i}
$$

注意到 $g, f$ 的递推式均为半在线卷积的形式，直接 CDQ 分治即可。

（半在线卷积时，考虑的是左半区间 $f$ 对右半区间 $f$ 的影响，还请注意甄别，将包含 $f$ 的项放于左侧）

求答案时，只需要将排列 $a$ 进行分段，每一个长度为 $l$ 加上对应的 $f_l + [l > 1]$ 即可。

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

/*
    多项式全家桶（2025.10.06）
    C++23 以上：请放心使用
    C++17 ~ C++20：
        请删去所有 poly 中的 constexpr（在上方 #define constexpr 即可）
    C++14：
        请将 RootPower 前面的 constexpr 改成 const
    C++11：
        请删去所有函数体前面的 constexpr，
        请将 findRootPower 与 RootPower 前面的 auto 改成 std::array<int, 32>
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
        std::vector<int> v(m);
        v[1] = 1;
        for (int i = 2; i < m; i ++) {
            v[i] = 1ll * v[mod % i] * (mod - mod / i) % mod;
        }

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

const int inf = 0x3f3f3f3f;

const int N = 100100;

int n;
int seq[N];

int fact[N], facv[N];

int f[N], g[N];

void solve_g(int l, int r) {
    if (l == r) {
        if (l == 1) {
            g[1] = 1;
        } else {
            g[l] = norm(fact[l] - g[l]);
        }
        return;
    }

    int mid = (l + r) >> 1;

    solve_g(l, mid);

    poly A1(mid - l + 1), B1(r - l + 1);
    for (int i = l; i <= mid; i ++) {
        A1[i - l] = g[i];
    }
    for (int i = 0; i <= r - l; i ++) {
        B1[i] = fact[i];
    }

    poly C1 = A1 * B1;
    for (int i = mid + 1; i <= r; i ++) {
        add(g[i], C1[i - l]);
    }

    solve_g(mid + 1, r);
}

void solve_f(int l, int r) {
    if (l == r) {
        if (l == 1) {
            f[1] = 0;
        } else {
            add(f[l], g[l]);
            mul(f[l], norm(qpow(fact[l] - g[l], mod - 2, mod)));
        }
        return;
    }

    int mid = (l + r) >> 1;

    solve_f(l, mid);

    poly A2(mid - l + 1), B2(r - l + 1);
    for (int i = l; i <= mid; i ++) {
        A2[i - l] = 1ll * g[i] * (f[i] + (i > 1)) % mod;
    }
    for (int i = 0; i <= r - l; i ++) {
        B2[i] = fact[i];
    }

    poly C2 = A2 * B2;
    for (int i = mid + 1; i <= r; i ++) {
        add(f[i], C2[i - l]);
    }

    poly A3(mid - l + 1), B3(r - l + 1);
    for (int i = l; i <= mid; i ++) {
        A3[i - l] = 1ll * fact[i] * f[i] % mod;
    }
    for (int i = 0; i <= r - l; i ++) {
        B3[i] = g[i];
    }

    poly C3 = A3 * B3;
    for (int i = mid + 1; i <= r; i ++) {
        add(f[i], C3[i - l]);
    }

    solve_f(mid + 1, r);
}

void prework() {
    fact[0] = 1;
    for (int i = 1; i <= n; i ++) {
        fact[i] = 1ll * fact[i - 1] * i % mod;
    }

    facv[n] = qpow(fact[n], mod - 2, mod);
    for (int i = n - 1; i >= 0; i --) {
        facv[i] = 1ll * facv[i + 1] * (i + 1) % mod;
    }

    solve_g(1, n);
    solve_f(1, n);

    // g[1] = 1;
    // for (int i = 2; i <= n; i ++) {
    //     g[i] = fact[i];
    //     for (int j = 1; j < i; j ++) {
    //         dec(g[i], 1ll * g[j] * fact[i - j] % mod);
    //     }
    // }

    // f[1] = 0;
    // for (int i = 2; i <= n; i ++) {
    //     f[i] = g[i];
    //     for (int j = 1; j < i; j ++) {
    //         add(f[i], 1ll * fact[i - j] * g[j] % mod * (f[j] + (j > 1) + f[i - j]) % mod);
    //     }
    //     mul(f[i], norm(qpow(fact[i] - g[i], mod - 2, mod)));
    // }

    // for (int i = 1; i <= n; i ++) {
    //     std::cout << ' ' << f[i];
    // }
    // std::cout << '\n';
}

int main() {
    std::ios::sync_with_stdio(0);
    std::cin.tie(0);

    std::cin >> n;
    
    for (int i = 1; i <= n; i ++) {
        std::cin >> seq[i];
    }

    prework();

    int ans = 0;
    for (int l = 1, r; l <= n; l = r + 1) {
        r = l;

        int maxv = -inf, minv = +inf;
        while (1) {
            chmax(maxv, seq[r]), chmin(minv, seq[r]);
            if (l == minv && r == maxv) {
                break;
            }
            r ++;
        }

        add(ans, f[r - l + 1]);
        if (l < r) {
            add(ans, 1);
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