---
layout: post
title:  "Basics on Dynamic Programming(DP) 9 - Backpack"
date:   2016-03-22 17:30:00
tags: [algorithm, leetcode, dynamic programming, dp, backpack]
categories: Algorithm
---

> backpack 背包问题

### 1. 0-1背包问题
{% highlight C++ %}
vector<int> dp(M+1, 0);  // 手里奖券数，能买的最大的价值
for(int i=0; i<N; i++) { //在执行i次循环后(此时已经处理i个物品)，前i个物体放到容量v时的最大价值，即之前的f[i][v]
    for(int j=M; j>=need[i]; j--)  // 现在有的钱
        dp[j] = max(dp[j], dp[j-need[i]]+val[i]);  // 不选，或选的最大值
}
cout << dp[M];
{% endhighlight %}
TODO 背包相关：
[背包问题九讲](http://love-oriented.com/pack/)
[backpack](http://www.lintcode.com/zh-cn/problem/backpack/)
[minimum-adjustment-cost](http://www.lintcode.com/zh-cn/problem/minimum-adjustment-cost/)
[k-sum](http://www.lintcode.com/zh-cn/problem/k-sum/)
