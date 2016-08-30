---
title: 388 Longest Absolute File Path
categories:
  - leetcode
date: 2016-08-30 22:43:00
tags:
---

> https://leetcode.com/problems/longest-absolute-file-path/

# 解析
直接模拟

# c++ Solution
```c++
class Solution {
   public:
    int lengthLongestPath(string input) {
        input += '\n';
        int pathPartLength[100];
        int tab = 0;
        bool append = false;
        bool isFile = false;
        int maxLength = 0;
        for (string::iterator it = input.begin(); it != input.end(); ++it) {
            switch (*it) {
                case '\n': {
                    if (isFile) {
                        int curLength = 0;
                        for (int i = 0; i <= tab; ++i) {
                            curLength += pathPartLength[i];
                        }
                        curLength += tab;
                        if (curLength > maxLength) {
                            maxLength = curLength;
                        }
                    }
                    tab = 0;
                    append = false;
                    isFile = false;
                    break;
                }
                case '\t': {
                    tab += 1;
                    break;
                }
                case '.': {
                    isFile = true;
                }
                default: {
                    if (!append) {
                        pathPartLength[tab] = 0;
                        append = true;
                    }
                    pathPartLength[tab] += 1;
                }
            }
        }
        return maxLength;
    }
};
```
