---
title: 4 Median of Two Sorted Arrays
categories:
  - leetcode
date: 2016-09-04 23:42:38
tags:
---

> https://leetcode.com/problems/median-of-two-sorted-arrays/

# 解析

1. 比较两个序列，将第一个数较小的序列交换到第一个序列，另外一个作为第二个序列
2. 通过二分法从第一个序列找出比第二个序列第一个数还小的一串数，把这串数从序列中截去
3. 重复1和2，在这个过程中找出排在两个序列中间的两个数（两个序列总长度为奇数时，这两个数是同一个数），计算这两个数的平均值

# C Solution
```c
double findMedianSortedArrays(int* nums1, int nums1Size, int* nums2, int nums2Size) {
    int mid1 = 0, mid2 = 0;
    int mid1Index, mid2Index;
    int dropIndex;
    int diff;
    int *tmpNums, tmpNumsSize;

    if (((nums1Size + nums2Size) & 1) == 0) {
        mid1Index = (nums1Size + nums2Size - 1) / 2;
        mid2Index = mid1Index + 1;
    } else {
        mid1Index = mid2Index = (nums1Size + nums2Size - 1) / 2;
    }

    while (1) {
        if (nums1Size == 0) {
            if (mid1Index >= 0) {
                mid1 = nums2[mid1Index];
            }
            mid2 = nums2[mid2Index];
            break;
        }

        if (nums2[0] < nums1[0]) {
            tmpNums = nums1;
            nums1 = nums2;
            nums2 = tmpNums;
            tmpNumsSize = nums1Size;
            nums1Size = nums2Size;
            nums2Size = tmpNumsSize;
        }

        dropIndex = nums1Size - 1;
        while (dropIndex > 0) {
            if (nums1[dropIndex] <= nums2[0]) {
                break;
            }
            dropIndex >>= 1;
        }
        if (mid1Index >= 0 && mid1Index <= dropIndex) {
            mid1 = nums1[mid1Index];
        }
        if (mid2Index >= 0 && mid2Index <= dropIndex) {
            mid2 = nums1[mid2Index];
            break;
        }
        diff = dropIndex + 1;
        mid1Index -= diff;
        mid2Index -= diff;
        nums1 += diff;
        nums1Size -= diff;
    }

    return (mid1 + mid2) / 2.0;
}
```
