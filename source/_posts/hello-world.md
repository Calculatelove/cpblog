---
title: 博客落地啦！
date: 2025-08-28 20:00:00
updated: 2025-08-30 20:00:00
---

博客做好了初步的工作！建立该博客的初衷详见[关于](/about)。下面是 KaTeX 公式测试：

<!-- more -->

行内公式：$\sum\limits_{k} \binom{s}{k}\binom{t}{m - k} = \binom{s + t}{m}$

行间公式：

$$
\det M = \sum\limits_{F : A \to B} (-1)^{\tau(F)} \prod\limits_{i = 1}^n \omega(P_i)
$$

矩阵：

$$
M =
\begin{bmatrix}
e(a_1, b_1) & e(a_1, b_2) & \cdots & e(a_1, b_n) \\
e(a_2, b_1) & e(a_2, b_2) & \cdots & e(a_2, b_n) \\
\vdots & \vdots & \ddots & \vdots \\
e(a_n, b_1) & e(a_n, b_2) & \cdots & e(a_n, b_n) \\
\end{bmatrix}
$$

多行公式：

$$
\begin{aligned}
& \sum\limits_{i = 1}^{n - 1} A_{n - 1 - s}^{i - 1} \cdot s \cdot (n - 1 - i)! \cdot (n - i) \\
= & \sum\limits_{i = 1}^{n - 1} \frac{(n - 1 - s)!}{(n - i - s)!} \cdot s \cdot (n - i)! \\
= & (n - 1 - s)! \cdot s \cdot \sum\limits_{i = 1}^{n - 1} \frac{(n - i)!}{(n - i - s)!} \\
= & (n - 1 - s)! \cdot s \cdot s! \cdot \sum\limits_{i = 1}^{n - 1} \frac{(n - i)!}{(n - i - s)! \cdot s!} \\
= & (n - 1 - s)! \cdot s \cdot s! \cdot \sum\limits_{i = 1}^{n - 1} \binom{n - i}{s} \\
= & (n - 1 - s)! \cdot s \cdot s! \cdot \sum\limits_{i = 1}^{n - 1} \binom{i}{s} \\
= & (n - 1 - s)! \cdot s \cdot s! \cdot \binom{n}{s + 1} \\
= & (n - 1 - s)! \cdot s \cdot s! \cdot \frac{n!}{(n - 1 - s)! \cdot (s + 1)!} \\
= & \frac{s}{s + 1} \cdot n!
\end{aligned}
$$