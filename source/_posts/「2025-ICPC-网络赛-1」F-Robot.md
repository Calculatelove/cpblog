---
title: 「2025 ICPC 网络赛 1」F. Robot
date: 2025-09-08 15:22:51
updated: 2025-09-08 15:22:51
categories: ICPC
tags:
  - 交互题
  - 构造
---

# Description

Link：[QOJ 14306](https://qoj.ac/contest/2513/problem/14306)

{% note default %}

**This is an interactive problem.**

一个机器人在一个无限大的二维平面的第一象限内移动。最初机器人位于坐标 $(s_x, s_y)$。最初所有单元格都是白色的。

游戏包含了若干次操作，最多 $T = 1000$ 次操作。每次操作都会按照以下顺序发生事件：
1. **你的操作**：你可以选择一个整点坐标 $(x, y)$，其中 $1 \leq x, y \leq T$，你会将该单元格标记成黑色。
2. **机器人的操作**：机器人可以移动到与其所在单元格八连通的单元格（由交互器决策）。
3. **结果检查**：机器人移动到下一个单元格之后，系统会检查该单元格是否是黑色。
  - 如果是黑色，机器人爆炸。
  - 如果是白色，则进行下一次操作。

你的目标是在 $T$ 次操作之内使机器人爆炸。

数据范围 $1 \leq s_x, s_y \leq 20$。

时空限制：$2$s / $1024$MiB。

{% endnote %}

<!-- more -->

# Solution

基础的想法是，**先构造一个正方形边界将机器人框住，再对框住的矩形内部进行查杀**。

尝试构造一个 $100 \times 100$ 的正方形边界。发现我们一开始不能将所有的边界都填充过去，这样机器人会在我们填充边界的过程中提前逃出边界。所以考虑先构造边界的一部分。

具体的，**先将右上角 $(100, 100)$ 填充，然后以右上角为起点，向左和向下每间隔 $2$ 个单元格填充一个黑点**。这样，等到机器人想要从某个缺口突破时，我们再将这个缺口填充。同时，**右上角的部分，向上和向右有两个缺口，我们先将这两个缺口都填充了**。此时只用了 $71$ 步。机器人最远只能逃到 $(91, 91)$。

接下来依次将边界填充完整。但是在机器人将要从一个缺口突破时，需要优先将这个缺口填充。具体地，对于向上的方向，机器人可能会从 $(x, 100), (x - 1, 100), (x + 1, 100)$ 这三个位置进行突破，于是当 $y \geq 97$ 时，我们就要依次判断这三个位置是否需要堵住了，恰好留有三步的容错。向右同理（由于已经将最右上角的两个缺口填充，所以向上和向右是互不干扰的）。

将边界填充完整之后，考虑对框住的矩形内部进行查杀。我们大可不必 $99 \times 99$ 地毯式查杀。考虑每次对于框住的矩形区域，取出最长的一条边，填充这条边的中线。然后对剩下的矩形继续进行操作，直到只剩下 $1\times 1$ 的矩形为止。设边长为 $L$，则所需要的步数为 $L + \frac{L}{2} + \frac{L}{2} + \frac{L}{4} + \frac{L}{4} + \dots = \mathcal{O}(L)$。

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

int px, py;

int vis[1001][1001];

int color(int x, int y) {
    if (x < 1 || x > 1000 || y < 1 || y > 1000) return 0;
    if (vis[x][y]) return 0;

    vis[x][y] = 1;
    std::cout << x << ' ' << y << std::endl;

    std::cin >> px >> py; if (!px) exit(0);
    return 1;
}

int main() {
    std::ios::sync_with_stdio(0);
    std::cin.tie(0);

    std::cin >> px >> py; if (!px) exit(0);

    color(100, 100);
    
    for (int i = 97; i >= 1; i -= 3) {
        color(i, 100);
        color(100, i);
    }

    color(99, 100);
    color(98, 100);
    color(100, 99);
    color(100, 98);

    for (;;) {
        if (px >= 97) {
            if (color(100, py)) continue;
            if (color(100, py + 1)) continue;
            if (color(100, py - 1)) continue;
        }
        if (py >= 97) {
            if (color(px, 100)) continue;
            if (color(px + 1, 100)) continue;
            if (color(px - 1, 100)) continue;
        }

        int flag = 1;
        for (int i = 1; i <= 100; i ++) {
            if (color(i, 100)) { flag = 0; break; }
            if (color(100, i)) { flag = 0; break; }
        }

        if (flag) break;
    }

    int sx = 1, ex = 99, sy = 1, ey = 99;
    while (sx < ex || sy < ey) {
        if (ex - sx > ey - sy) {
            int mid = (sx + ex) >> 1;
            for (int j = sy; j <= ey; j ++) {
                color(mid, j);
            }
            if (px < mid) {
                ex = mid - 1;
            } else {
                sx = mid + 1;
            }
        } else {
            int mid = (sy + ey) >> 1;
            for (int i = sx; i <= ex; i ++) {
                color(i, mid);
            }
            if (py < mid) {
                ey = mid - 1;
            } else {
                sy = mid + 1;
            }
        }
    }

    color(sx, sy);

    return 0;
}

/**
 *  心中无女人
 *  比赛自然神
 *  模板第一页
 *  忘掉心上人
**/
```