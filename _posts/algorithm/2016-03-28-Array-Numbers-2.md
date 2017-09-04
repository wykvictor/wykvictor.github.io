---
layout: post
title:  "Array & Numbers 2 - Sub Array"
date:   2016-03-28 10:30:00
tags: [algorithm, leetcode, array, subarray]
categories: Algorithm
---

> Sub Array: 解法 Prefix Sum

> 二维matrix: sum[i,j->x,y] = sum(x,y)-sum(x,j-1)-sum(i-1,y)+sum(i-1,j-1)

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
You may complete at most k transactions. Hard!
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

### 2. [Maximum Subarray](http://www.lintcode.com/en/problem/maximum-subarray/)
```
Given the array [−2,2,−3,4,−1,2,1,−5,3], the contiguous subarray [4,−1,2,1] has the largest sum = 6.
```
{% highlight C++ %}
// 同1是同一个问题prefix[j]-prefix[i-1]，是i~j的subarray的sum
int maxSubArray(vector<int> nums) {
  if (nums.size() == 0) return 0;
  int prefix = 0, minpre = 0, res = INT_MIN;
  for (int i = 0; i < nums.size(); i++) {
    prefix += nums[i];
    res = max(res, prefix - minpre);
    minpre = min(minpre, prefix);  // 每次记录minpre
  }
  return res;
}
{% endhighlight %}
另方法，之前的思路
{% highlight C++ %}
int maxSubArray(vector<int> nums) {
  if (nums.size() == 0) return 0;
  int presum = nums[0], res = nums[0];
  for (int i = 1; i < nums.size(); i++) {
    if (presum <= 0) {  // 若之前是小于0的，则丢掉，对结果没有好处
      presum = nums[i];
    } else {
      presum += nums[i];
    }
    res = max(res, presum);
  }
  return res;
}
{% endhighlight %}

### 3. [Maximum Subarray II](http://www.lintcode.com/en/problem/maximum-subarray-ii/)
```
Find two non-overlapping subarrays which have the largest sum.
```
{% highlight C++ %}
// 和sell stock III非常相似
int maxTwoSubArrays(vector<int> nums) {
  int N = nums.size();
  if (N == 0) return 0;
  // left to right
  vector<int> pre(N, INT_MIN);  // 目前为止，前边最大前缀和
  int minpre = 0, presum = 0;
  for (int i = 0; i < N; i++) {
    presum += nums[i];
    pre[i] = max(i == 0 ? INT_MIN : pre[i - 1], presum - minpre);
    minpre = min(minpre, presum);
  }
  // right to left
  vector<int> post(N, INT_MIN);
  int minpost = 0, postsum = 0;
  for (int i = N - 1; i >= 0; i--) {
    postsum += nums[i];
    post[i] = max(i == N - 1 ? INT_MIN : post[i + 1], postsum - minpost);
    minpost = min(minpost, postsum);
  }
  // get result
  int res = INT_MIN;
  for (int i = 0; i < N - 1; i++) {
    res = max(res, pre[i] + post[i + 1]);
  }
  return res;
}
{% endhighlight %}

### 4. [Maximum Subarray III](http://www.lintcode.com/en/problem/maximum-subarray-iii/)
```
Find K non-overlapping subarrays which have the largest sum.
TODO: DP, hard
```

### 5. [Minimum Subarray](http://www.lintcode.com/en/problem/minimum-subarray/)
方法1：全部取负数，再调用2中函数；方法2，直接按同样思路算
{% highlight C++ %}
int minSubArray(vector<int> nums) {
  if (nums.size() == 0) return 0;
  int pre = 0, maxpre = 0, res = INT_MAX;
  for (auto i : nums) {
    pre += i;
    res = min(res, pre - maxpre);
    maxpre = max(maxpre, pre);
  }
  return res;
}
{% endhighlight %}

### 6. [Maximum Subarray Difference](http://www.lintcode.com/en/problem/maximum-subarray-difference/)
```
Find two non-overlapping subarrays A and B, which |SUM(A) - SUM(B)| is the largest.
```
{% highlight C++ %}
// 前边几道题的集大成
int maxDiffSubArrays(vector<int> nums) {
  int N = nums.size();
  if (N == 0) return 0;
  // left2right
  vector<int> maxleft(N, INT_MIN);  // 左边subarray最大和
  int premin = 0, presum = 0;
  for (int i = 0; i < N; i++) {
    presum += nums[i];
    maxleft[i] = max(i == 0 ? INT_MIN : maxleft[i - 1], presum - premin);
    premin = min(premin, presum);
  }
  vector<int> minleft(N, INT_MAX);
  int premax = 0;
  presum = 0;
  for (int i = 0; i < N; i++) {  // 左边subarray最小和
    presum += nums[i];
    minleft[i] = min(i == 0 ? INT_MAX : minleft[i - 1], presum - premax);
    premax = max(premax, presum);
  }
  // right2left
  vector<int> maxright(N, INT_MIN);
  int postmin = 0, postsum = 0;
  for (int i = N - 1; i >= 0; i--) {  // 右边subarray最大和
    postsum += nums[i];
    maxright[i] = max(i == N - 1 ? INT_MIN : maxright[i + 1], postsum - postmin);
    postmin = min(postmin, postsum);
  }
  vector<int> minright(N, INT_MAX);
  int postmax = 0;
  postsum = 0;
  for (int i = N - 1; i >= 0; i--) {  // 右边subarray最小和
    postsum += nums[i];
    minright[i] = min(i == N - 1 ? INT_MAX : minright[i + 1], postsum - postmax);
    postmax = max(postmax, postsum);
  }
  // calculate result
  int res;
  for (int i = 0; i < N - 1; i++) {
    res = max(res, abs(maxleft[i] - minright[i + 1]));
    res = max(res, abs(minleft[i] - maxright[i + 1]));
  }
  return res;
}
{% endhighlight %}

### 7. [Subarray Sum](http://www.lintcode.com/en/problem/subarray-sum/)
```
Find a subarray where the sum of numbers is zero. 
Given [-3, 1, 2, -3, 4], return [0, 2] or [1, 3]
```
{% highlight C++ %}
vector<int> subarraySum(vector<int> nums) {
  vector<int> res;
  unordered_map<int, int> hash;
  hash[0] = -1;  // -1代表0以前的
  int presum = 0;
  for (int i = 0; i < nums.size(); i++) {
    presum += nums[i];                      // inclusive!
    if (hash.find(presum) != hash.end()) {  // 找到相同的和
      res.push_back(hash[presum] + 1);
      res.push_back(i);
      return res;
    }
    hash[presum] = i;
  }
  return res;
}
{% endhighlight %}

### 8. [Subarray Sum Closest](http://www.lintcode.com/en/problem/subarray-sum-closest/)
```
Find a subarray with sum closest to zero. Return the indexes of the first number and last number.
```
{% highlight C++ %}
vector<int> subarraySumClosest(vector<int> nums) {
  if (nums.size() == 0) return vector<int>();
  vector<int> res(2, 0);
  if (nums.size() == 1) return res;

  map<int, int> hash;
  hash[0] = -1;  // -1代表0以前的 下标
  int presum = 0;
  for (int i = 0; i < nums.size(); i++) {
    presum += nums[i];                      // inclusive!
    if (hash.find(presum) != hash.end()) {  // 加速：找到相同的和,立马退出
      res[0] = hash[presum] + 1;
      res[1] = i;
      return res;
    }
    hash[presum] = i;
  }
  // 跟上题唯一区别：没有等于0的，就排序选最小的
  // map自动排序
  map<int, int>::iterator it = hash.begin();
  int mingap = INT_MAX, lastSum = it->first, lastIndex = it->second;
  for (it++; it != hash.end(); it++) {
    if (it->first - lastSum < mingap) {
      res[0] = min(it->second, lastIndex) + 1;  // exclusive!
      res[1] = max(it->second, lastIndex);
      mingap = it->first - lastSum;
    }
    lastSum = it->first;
    lastIndex = it->second;  // 别忘了更新last
  }
  return res;
}
{% endhighlight %}
