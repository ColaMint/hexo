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
ass Solution {
   public:
    int lengthLongestPath(string input) {
        input += '\n';
        string* paths[100];
        int tab = 0;
        int maxTab = -1;
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
                    if (tab > maxTab) {
                        maxTab = tab;
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
        for (int i = 0; i <= maxTab; ++i) {
            delete paths[i];
        }
        return maxLength;
    }
};
```
