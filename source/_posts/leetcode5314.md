---
title: BFS - LeetCode 跳跃游戏IV
date: 2020-02-09 21:32:58
mathjax: true
categories:
- 数据结构与算法
tags:
- BFS
---

# 跳跃游戏 IV

周赛没写出来的题，[题目](https://leetcode-cn.com/problems/jump-game-iv/)如下:

给你一个整数数组 arr ，你一开始在数组的第一个元素处（下标为 0）。每一步，你可以从下标 i 跳到下标：

1. `i + 1 满足：i + 1 < arr.length`
2. `i - 1 满足：i - 1 >= 0`
3. `j 满足：arr[i] == arr[j] 且 i != j`
    请你返回到达数组最后一个元素的下标处所需的 最少操作次数 。

注意：任何时候你都不能跳到数组外面。
<!--more-->

**示例：**

```c
输入：arr = [100,-23,-23,404,100,23,23,23,3,404]
输出：3
解释：那你需要跳跃 3 次，下标依次为 0 --> 4 --> 3 --> 9 。下标 9 为数组的最后一个元素的下标。
```

**数据范围：**

- `1 <= arr.length <= 5 * 10^4`
- `-10^8 <= arr[i] <= 10^8`

这题可以用BFS解决，但是复杂度为$o(n^2)$，需要进行优化。优化的方法是：不连续使用方式3跳两次，因为使用方式3跳两次可以用直接跳一次代替。

具体的例子是：对于`arr = [1, 7_1, 7_2, 7_3, 7_4, 7_5, 2]`（把相同的`7`记为`7_1, 7_2, 7_3, 7_4, 7_5`），操作次数最少的操作为：`1 -> 7_1 -> 7_5 -> 2`。在以`7_1`为起点考虑过`7_1 -> 7_2`、`7_1 -> 7_3`、`7_1 -> 7_4`、`7_1 -> 7_5`之后，不再使用方式3在`7`之间跳转。

具体代码如下：

```c++
class Solution {
public:
    unordered_map<int, vector<int>> bucket;
    int minJumps(vector<int>& arr) {
        int len  = arr.size();
        for(int i = 0; i < len; ++i) {
            bucket[arr[i]].push_back(i);
        }

        int book[50005];
        memset(book, -1, sizeof(book));
        queue<int> que;
        que.push(0);
        book[0] = 0;
        while (!que.empty()) {
            int temp = que.front();
            que.pop();
            if (temp == len - 1) {
                break;
            }
            int nextIdx;
            // 方式1
            if (temp + 1 < len) {
                nextIdx = temp + 1;
                if (book[nextIdx] == -1) {
                    book[nextIdx] = book[temp] + 1;
                    que.push(nextIdx);
                }
            }
            // 方式2
            if (temp - 1 >= 0) {
                nextIdx = temp - 1;
                if (book[nextIdx] == -1) {
                    book[nextIdx] = book[temp] + 1;
                    que.push(nextIdx);
                }
            }
            // 方式3
            vector<int> &vc = bucket[arr[temp]];
            for (auto nextIdx : vc) {
                if (book[nextIdx] == -1) {
                    book[nextIdx] = book[temp] + 1;
                    que.push(nextIdx);
                }
            }
            vc.clear(); //方式3中遍历过bucket[arr[temp]]后，将其清空
        }
        return book[len - 1];
    }
};
```