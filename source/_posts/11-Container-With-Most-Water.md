---
title: 11 Container With Most Water
categories:
  - leetcode
date: 2016-09-17 21:17:19
tags:
---

> https://leetcode.com/submissions/detail/74643962/

# 解析

设置两个指针`i`和`j`, `i`从左边开始, `j`从右边开始

当`height[i]` < `height[j]`:

* 若向左移动`j`, `i`和`j`的距离将缩短，`height[j]`增大对增大面积没有帮助，反而当`height[j]` < `height[i]`时, 面积会更小, 因此面积不可能增大
* 若向右移动`i`, `i`和`j`的距离将缩短，但是如果`height[i]`增大的话，有可能使面积增大

当`height[i]` > `height[j]`时推理类似

结论：向内移动较短一边可能使面积增大


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
        curArea = calArea(height, l, r);
        if (curArea > maxArea) {
            maxArea = curArea;
        }
    }
    return maxArea;
}
```
