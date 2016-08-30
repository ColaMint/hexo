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
        string** paths = new string*[100];
        int tab = 0;
        bool append = false;
        int maxLength = 0;
        for (string::iterator it = input.begin(); it != input.end(); ++it) {
            char c = *it;
            switch (c) {
                case '\n': {
                    if (paths[tab]->find('.', 0) != string::npos) {
                        int curLength = 0;
                        for (int i = 0; i <= tab; ++i) {
                            curLength += paths[i]->length();
                        }
                        curLength += tab;
                        if (curLength > maxLength) {
                            maxLength = curLength;
                        }
                    }
                    tab = 0;
                    append = false;
                    break;
                }
                case '\t': {
                    tab += 1;
                    break;
                }
                default: {
                    if (!append) {
                        paths[tab] = new string();
                        append = true;
                    }
                    *paths[tab] += c;
                }
            }
        }
        return maxLength;
    }
};
```
