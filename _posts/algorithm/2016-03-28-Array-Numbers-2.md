---
layout: post
title:  "Array & Numbers 2 - Sub Array"
date:   2016-03-28 10:30:00
tags: [algorithm, leetcode, array, subarray]
categories: Algorithm
---

> Sub Array: 解法 Prefix Sum

### 1. [Best Time to Buy and Sell Stock](http://www.lintcode.com/en/problem/best-time-to-buy-and-sell-stock/)
```
If you were only permitted to complete at most one transaction (ie, buy one and sell one share of the stock),
design an algorithm to find the maximum profit.
```
{% highlight C++ %}
int maxProfit(vector<int> &prices) {
  if (prices.empty()) return 0;
  int res = 0, premin = prices[0];
  for (int i = 1; i < prices.size(); i++) {
    res = max(res, prices[i] - premin);
    premin = min(premin, prices[i]);
  }
  return res;
}
{% endhighlight %}
```
You may complete as many transactions as you like. However, you may not engage in multiple transactions at the same time (ie, you must sell the stock before you buy again).
```
{% highlight C++ %}
// 问题简化为：Greedy，累加所有上升区间
int maxProfit(vector<int> &prices) {
  int res = 0;
  for (int i = 1; i < prices.size(); i++) {
    if (prices[i] > prices[i - 1]) res += prices[i] - prices[i - 1];
  }
  return res;
}
{% endhighlight %}
```
Important! You may complete at most two transactions.
```
{% highlight C++ %}
int maxProfit(vector<int> &prices) {
  int n = prices.size();
  if (n <= 1) return 0;

  // 从左往右扫一遍，记录prefix交易的最大利润
  vector<int> pre(n, 0);  // !! 注意，这里是0
  int premin = prices[0];
  for (int i = 1; i < n; i++) {
    pre[i] = max(pre[i - 1], prices[i] - premin);
    premin = min(premin, prices[i]);
  }
  // 从右往左扫一遍，记录postfix天交易
  vector<int> post(n, 0);
  int postmax = prices[n - 1];
  for (int i = n - 2; i >= 0; i--) {
    post[i] = max(post[i + 1], postmax - prices[i]);
    postmax = max(postmax, prices[i]);
  }
  // 开始扫一遍，算
  int res = 0;
  for (int i = 1; i < n; i++) {
    res = max(res, pre[i] + post[i]);  //注意，其实可以当天卖，当天买。相当于做一笔生意了呗!
  }
  return res;
}
{% endhighlight %}
```
You may complete at most k transactions.
```
{% highlight C++ %}
int maxProfit(int k, vector<int> &prices) {
  if (k == 0) return 0;
  if (k >= prices.size() / 2) {
    int res = 0;
    for (int i = 1; i < prices.size(); i++) {
      if (prices[i] > prices[i - 1]) res += prices[i] - prices[i - 1];
    }
    return res;
  }
  // dp[i][k] 表示前i天，至多进行k次交易，第k天可以不sell的最大获益
  // dpsell[i][k] 表示前i天，至多进行k次交易，第k天必须sell的最大获益
  int N = prices.size();
  vector<vector<int> > dp(N, vector<int>(k + 1, 0));  // 初始化为0
  vector<vector<int> > dpsell(N, vector<int>(k + 1, 0));
  for (int i = 1; i < N; i++) {
    int increase = prices[i] - prices[i - 1];
    dpsell[i][0] = 0;
    for (int j = 1; j <= k; j++) {
      dpsell[i][j] =
          max(dp[i - 1][j - 1] + increase, dpsell[i - 1][j] + increase);
      dp[i][j] = max(dp[i - 1][j], dpsell[i][j]);  // 在第j天卖出去中，选最大的
    }
  }
  return dp[N-1][k];
}
{% endhighlight %}