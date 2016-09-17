---
title: 11 Container With Most Water
categories:
  - leetcode
date: 2016-09-17 21:17:19
tags:
---

> https://leetcode.com/submissions/detail/74643962/

# 解析

采用贪心算法

每次往内重新选取较短的一条边，期望得到更大的有效面积


# c solution
```c
int calArea(int* height, int l, int r) {
    int ly = height[l];
    int ry = height[r];
    return ly < ry ? (r - l) * ly : (r - l) * ry;
}

int maxArea(int* height, int heightSize) {
    int l = 0;
    int r = heightSize - 1;
    int maxArea = calArea(height, l, r);
    int curArea;
    while (l < r - 1) {
        if (height[l] < height[r]) {
            ++l;
        } else {
            --r;
        }
        printf("%d %d\n", l, r);
        curArea = calArea(height, l, r);
        if (curArea > maxArea) {
            maxArea = curArea;
        }
    }
    return maxArea;
}
```
