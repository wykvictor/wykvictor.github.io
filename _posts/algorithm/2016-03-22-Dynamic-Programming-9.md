---
layout: post
title:  "Basics on Dynamic Programming(DP) 9 - Backpack"
date:   2016-03-22 17:30:00
tags: [algorithm, leetcode, dynamic programming, dp, backpack]
categories: Algorithm
---

> backpack 背包问题

*  [背包问题九讲](http://love-oriented.com/pack/)
*  [backpack](http://www.lintcode.com/zh-cn/problem/backpack/)
*  [minimum-adjustment-cost](http://www.lintcode.com/zh-cn/problem/minimum-adjustment-cost/)
*  [k-sum](http://www.lintcode.com/zh-cn/problem/k-sum/)

### 1. 0-1背包问题
{% highlight C++ %}
vector<int> dp(M+1, 0);  // 手里奖券数，能买的最大的价值
// 在执行i次循环后(此时已经处理i个物品)，前i个物体放到容量v时的最大价值，即之前的f[i][v]
for(int i=0; i<N; i++) {
  for(int j=M; j>=need[i]; j--)  // 现在有的钱
    dp[j] = max(dp[j], dp[j-need[i]]+val[i]);  // 不选，或选的最大值
}
cout << dp[M];
{% endhighlight %}

### 2. [硬币找零问题](http://acm.nyist.net/JudgeOnline/problem.php?pid=995)
```
人民币的硬币系统是 100，50，20，10，5，2，1，0.5，0.2，0.1，0.05，0.02，0.01 元，采用这些硬币我们可以对任何一个工资数用贪心算法求出其最少硬币数。 
但不幸的是：我们可能没有这样一种好的硬币系统，因此用贪心算法不能求出最少的硬币数，甚至有些金钱总数还不能用这些硬币找零。
例如，如果硬币系统是 40，30，25 元，那么 37元就不能用这些硬币找零；95元的最少找零硬币数是 3。
又如，硬币系统是 10，7，5，1元，那么 12 元用贪心法得到的硬币数为 3，而最少硬币数是 2。

你的任务就是：对于任意的硬币系统和一个金钱数，请你编程求出最少的找零硬币数；
如果不能用这些硬币找零，请给出一种找零方法，使剩下的钱最少(剩下钱数最少的找零方案中的最少硬币数。)。 
```

{% highlight C++ %}
// DP[i]表示i最少需要的硬币个数，假设给定的硬币面值为a,b,c,d……
// 则dp[i]=min(dp[i-a],dp[i-b],dp[i-c],dp[i-d],……)+1 (i-a,i-b,……>=0)
int Solve(vector<int> &num, int money) {
  int n = num.size();
  // 初始化dp
  vector<int> dp(money + 1, 100001);  // 初始化为题目给定的最大值
  dp[0] = 0;
  for (auto i : num) dp[i] = 1;  // 需要1个硬币

  sort(num.begin(), num.end());  // 优化算法，从小到大排序

  for (int i = 1; i <= money; i++) {              // 总钱数，开始循环
    for (int j = 0; num[j] <= i && j < n; j++) {  // 注意终止条件num[j] <= i
      dp[i] = min(dp[i], dp[i - num[j]] + 1);
    }
  }

  while (dp[money] == 100001)  //所给数据不能用硬币表示
    money--;
  return dp[money];
}
{% endhighlight %}