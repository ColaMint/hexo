---
title: 375 Guess Number Higher or Lower II
categories:
  - leetcode
date: 2016-08-28 19:26:00
tags:
---
> https://leetcode.com/problems/guess-number-higher-or-lower-ii/

# 解析
采用动态规划

用`dp[i][j]`表示：当游戏从`i <= x <= j`开始时，`至少`花费多少钱才能保证胜利

注意：当游戏只剩下一个数字时，直接结束，不用继续猜测

初始化：d[0][0], dp[1][1], ..., dp[n][n] = 0

动规过程：
以初始化的数据为基础，依次计算出
dp[0][1], dp[1][2], ..., dp[n-1][n],
dp[0][2], dp[1][3], ..., dp[n-2][n],
...
dp[0][n]
题目的结果即dp[0][n]的值

假设我们要计算dp[3][5]的值，我们下一步能猜测是数字有3, 4, 5
当我们猜3时，花费是 3 + dp[4][5]
当我们猜4时，花费是 4 + max(dp[3][3], dp[5][5])
当我们猜5时，花费是 dp[3][4] + 5
因此dp[3][5] = min(3 + dp[4][5],  4 + max(dp[3][3], dp[5][5]), dp[3][4] + 5)

为什么当我们猜测4时，式子中是max而不是min呢？
这是因为，题目要求的是至少花费多少钱才能保证胜利。
因此，当我们选择一个数之后，我们要继续走最坏的打算。

# C++ Solution
```c++
class Solution {
   public:
    int getMoneyAmount(int n) {
        int dp[500][500];
        for (int i = 0; i <= n; ++i) {
            dp[i][i] = 0;
        }
        for (int range = 1; range <= n; ++range) {
            for (int i = 0; i < n; ++i) {
                int j = i + range;
                if (j > n) {
                    break;
                }
                int min_cost = -1;
                for (int guess = i; guess <= j; ++guess) {
                    int left_cost = guess > i ? dp[i][guess - 1] : 0;
                    int right_cost = guess < j ? dp[guess + 1][j] : 0;
                    int cost = guess + (left_cost > right_cost ? left_cost
                                                               : right_cost);
                    if (min_cost == -1 || cost < min_cost) {
                        min_cost = cost;
                    }
                }
                dp[i][j] = min_cost;
            }
        }
        return dp[0][n];
    }
};
```
