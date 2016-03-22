---
layout: post
title:  "Basics on Dynamic Programming(DP) 7 - Interval DP"
date:   2016-03-20 16:30:00
tags: [algorithm, leetcode, dynamic programming, dp]
categories: Algorithm
---

> 区间内动态规划

### 1. Merge Stone
```
有n堆石子排成一列，每堆石子有一个重量w[i].
每次合并可以合并相邻的两堆石子，一次合并的代价为两堆石子的重量和w[i]+w[i+1]。
问安排怎样的合并顺序，能够使得总合并代价达到最小。
Example 1:
4 1 1 4
--- ---
 5   5 --- 10
 ----
  10   --- 10
        = 20
Example 2:
4 1 1 4
  ---
   2 
   ----
     6
------
  10  2 + 6+ 10 = 18
```

* state: f[i][j]代表了从i合并到j的最小代价是什么
* function: f[i][j] = min(f[i][k] + f[k+1][j] + sum[i][j])，sum是一定的，从k中找最小的
* intialize: f[i][i] = 0
* answer: f[1][n]
{% highlight C++ %}
for(int len=1; len<n; len++) {
    for(int i=1; i+len<=n; i++) {
        int end = i+len;
        int tmp = INF;
        int k = 0;
        for(int j=p[i][end-1]; j<=p[i+1][end]; j++) {
            if(dp[i][j] + dp[j+1][end] + sum[end] - sum[i-1] < tmp) {
                tmp = dp[i][j] + dp[j+1][end] + sum[end] - sum[i-1];
                k = j;
            }
        }
        dp[i][end] = tmp;
        p[i][end] = k;
    }
}
{% endhighlight %}
