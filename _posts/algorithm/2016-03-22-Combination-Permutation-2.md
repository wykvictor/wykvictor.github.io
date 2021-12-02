---
layout: post
title:  "Combination and Permutation 2 - Combination Sum"
date:   2016-03-22 22:30:00
tags: [algorithm, leetcode, DFS, combination sum]
categories: Algorithm
---

### 1. [Combination Sum - Leetcode 39](https://leetcode.com/problems/combination-sum/)
```
Given a set of candidate numbers (C) and a target number (T), find all unique combinations in C where the candidate numbers sums to T.
The same repeated number may be chosen from C unlimited number of times.
Note:
All numbers (including target) will be positive integers.
Elements in a combination (a1, a2, … , ak) must be in non-descending order. (ie, a1 ≤ a2 ≤ … ≤ ak).
The solution set must not contain duplicate combinations.
For example, given candidate set 2,3,6,7 and target 7, 
A solution set is: 
[7] 
[2, 2, 3]
```

利用通用模板:
{% highlight C++ %}
vector<vector<int> > combinationSum(vector<int> &candidates, int target) {
  // DFS 深搜
  sort(candidates.begin(), candidates.end());
  vector<vector<int> > res;
  vector<int> path;
  combinationSumCore(candidates, res, path, target, 0, 0);
  return res;  //!!别忘返回
}
void combinationSumCore(vector<int> &candidates, vector<vector<int> > &res,
                        vector<int> &path, int target, int start, int sum) {
  // if(sum > target)  加入优化后，不需要了！
  //    return;
  if (sum == target) {
    res.push_back(path);
    return;
  }
  for (int i = start; i < candidates.size(); i++) {
    if (sum + candidates[i] > target)
      break;  //剪枝优化：说明不用再向后走了，已经超了
    // 防止输入是[2 2 3]这种有重复的情况
    if (i != start && candidates[i] == candidates[i - 1]) continue;
    path.push_back(candidates[i]);
    // input same "i", because same repeated number is allowed
    combinationSumCore(candidates, res, path, target, i, sum + candidates[i]);
    path.pop_back();
  }
}
{% endhighlight %}

### 2. [Combination Sum II- Leetcode 40](https://leetcode.com/problems/combination-sum-ii/)
```
Given a collection of candidate numbers (C) and a target number (T), find all unique combinations in C where the candidate numbers sums to T.
Each number in C may only be used once in the combination.
Note:
All numbers (including target) will be positive integers.
Elements in a combination (a1, a2, … , ak) must be in non-descending order. (ie, a1 ≤ a2 ≤ … ≤ ak).
The solution set must not contain duplicate combinations.
For example, given candidate set 10,1,2,7,6,1,5 and target 8, 
A solution set is: 
[1, 7] 
[1, 2, 5] 
[2, 6] 
[1, 1, 6]
```

与上题区别，一个数字不能用多次，改Core中递归调用一行就OK,同时不能有duplicate的解：
{% highlight C++ %}
void combinationSum2Core(vector<int> &candidates, vector<vector<int> > &res,
                         vector<int> &path, int target, int start, int sum) {
  if (sum == target) {
    res.push_back(path);
    return;
  }
  for (int i = start; i < candidates.size(); i++) {
    if (sum + candidates[i] > target) break;
    //[1,1], 1 Output: [[1],[1]] Expected: [[1]]
    if (i != start && candidates[i] == candidates[i - 1]) continue;
    path.push_back(candidates[i]);
    // The only difference is i+1
    combinationSum2Core(candidates, res, path, target, i + 1,
                        sum + candidates[i]);
    path.pop_back();
  }
}
{% endhighlight %}

### 3. [Combination Sum III- Leetcode 216](https://leetcode.com/problems/combination-sum-iii/)
```
Find all possible combinations of k numbers that add up to a number n, given that only numbers from 1 to 9 can be used and each combination should be a unique set of numbers.

Ensure that numbers within the set are sorted in ascending order.

Example 1:
Input: k = 3, n = 7
Output:
[[1,2,4]]

Example 2:
Input: k = 3, n = 9
Output:
[[1,2,6], [1,3,5], [2,3,4]]
```

{% highlight C++ %}
void combineCore(vector<vector<int>> &res, vector<int> &path, int k, int n, int sum, int pos) {
    if(sum == n && path.size() == k) {  // 2个限制条件
        res.push_back(path);
        return;
    }
    for(int i=pos; i<=9; i++) {
        if(sum + pos > n || path.size() == k)  break;  // 优化
        path.push_back(i);
        combineCore(res, path, k, n, sum+i, i+1);
        path.pop_back();
    }
}

vector<vector<int>> combinationSum3(int k, int n) {
    vector<vector<int>> res;
    vector<int> path;
    combineCore(res, path, k, n, 0, 1);
    return res;
}
{% endhighlight %}
Java中，List是一个接口，ArrayList是一个类继承并实现了List
 