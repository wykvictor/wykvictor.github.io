---
layout: post
title:  "Combination and Permutation 1 - General Template"
date:   2016-03-22 20:30:00
tags: [algorithm, leetcode, DFS, combination, permutation]
categories: Algorithm
---

### 1. [Subsets - Leetcode 78](https://leetcode.com/problems/Subsets/)
```
Given a set of distinct integers, S, return all possible subsets.
Note:
Elements in a subset must be in non-descending order.
The solution set must not contain duplicate subsets.
For example,
If S = [1,2,3], a solution is:
[
  [3],
  [1],
  [2],
  [1,2,3],
  [1,3],
  [2,3],
  [1,2],
  []
]
```

通用模板： 时间复杂度 O(2^n)，空间复杂度 O(n)
{% highlight C++ %}
vector<vector<int> > subsets(vector<int> &S) {
  vector<vector<int> > res;
  if (S.empty()) return res;
  sort(S.begin(), S.end());  // could change S or not?
  vector<int> path;
  // Don't need for combination! vector<bool> visited(S.size(), false);
  subsetsCore(S, res, path, 0);
  return res;
}
void subsetsCore(vector<int> &S, vector<vector<int> > &res, vector<int> &path, int start) {
  res.push_back(path);
  for (int i = start; i < S.size(); i++) {
    path.push_back(S[i]);              // 选
    subsetsCore(S, res, path, i + 1);  // Caution: i+1
    path.pop_back();                   // 不选, 换下一个
  }
}
{% endhighlight %}
其他方法，二进制模拟：
{% highlight C++ %}
vector<vector<int> > subsets(vector<int> &S) {
  // Use binary to simulate.
  // S.size() limited to 32
  //时间复杂度 O(2^n)，空间复杂度 O(1)
  //这种方法最巧妙。因为它不仅能生成子集，还能方便的表示集合的并、交、差等集合运算
  //设两个集合的位向量分别为B和B，则B | B ;B & B ;B ^ B
  //分别对应集合的并、交、对称差
  vector<vector<int> > res;
  int len = S.size();
  if (len == 0) return res;
  sort(S.begin(), S.end());
  int times = 1 << len;
  for (int i = 0; i < times; i++) {
    vector<int> path;
    for (int j = 0; j < len; j++) {
            if(i & (1 << j))    // important，这一位是1就取出来
                path.push_back(S[j]);
    }
    res.push_back(path);
  }
}
{% endhighlight %}
增量构造法，迭代，简洁：iteration increment
{% highlight C++ %}
vector<vector<int> > subsets(vector<int> &S) {
  //增量迭代  时间复杂度 O(2^n)，空间复杂度 O(1)
  sort(S.begin(), S.end());
  vector<vector<int> > res(1);  //先开辟1个的空间,must has one element, the []
  for (int i = 0; i < S.size(); i++) {
    for (int j = res.size() - 1; j >= 0; j--) {
      vector<int> path = res[j];
      path.push_back(S[i]);
      res.push_back(path);
    }
  }
  return res;
}
{% endhighlight %}

### 2. [Subsets II - Leetcode 90](https://leetcode.com/problems/Subsets-ii/)
```
Given a collection of integers that might contain duplicates, S, return all possible subsets.
Note:
Elements in a subset must be in non-descending order.
The solution set must not contain duplicate subsets.
For example,
If S = [1,2,2], a solution is:
[
  [2],
  [1],
  [1,2,2],
  [2,2],
  [1,2],
  []
]
```

利用通用模板:
{% highlight C++ %}
vector<vector<int> > subsetsWithDup(vector<int> &S) {
  sort(S.begin(), S.end());
  vector<vector<int> > res;
  vector<int> path;
  subsetsWithDupCore(S, res, path, 0);
  return res;
}
void subsetsWithDupCore(vector<int> &S, vector<vector<int> > &res,
                        vector<int> &path, int start) {
  res.push_back(path);
  for (int i = start; i < S.size(); i++) {
    // not i!=0,应该是i!=start，只有第一次进去的时候，放入，
    // 其他的情况都是，放第一个2和放第二个2一样的，只有数量的区别。也就是在同一个循环内，避免放同样的数字
    if (i != start && S[i] == S[i - 1]) continue;
        path.push_back(S[i]);
        subsetsWithDupCore(S, res, path, i+1);
        path.pop_back();
    }
}
{% endhighlight %}
其他方法：

1，二进制模拟  不适用

可以内部去重判断，if(find(res.begin(), res.end(), line) == res.end())
这样也比较慢，没有下面的方法好

2，增量构造法，这个可以用，较难理解
{% highlight C++ %}
vector<vector<int> > subsetsWithDup(vector<int> &S) {
  vector<vector<int> > res(1);
  sort(S.begin(), S.end());
  int len = 1;
  int start = 1;  // record the same entrance
  for (int i = 0; i < S.size(); i++) {
    //相同的数字，那么start就从上次开始的地方开始，上次start之前的，不能重复弄了!
    if (i != 0 && S[i] == S[i - 1])
      start = len;
    else
      start = 0;
    len = res.size();  //更新len
    for (int j = start; j < len; j++) {
      vector<int> path = res[j];
      path.push_back(S[i]);
            res.push_back(path);
    }
  }
  return res;
}
{% endhighlight %}

### 3. [Combinations](https://leetcode.com/problems/Combinations/)
```
Given two integers n and k, return all possible combinations of k numbers out of 1 ... n.
For example,
If n = 4 and k = 2, a solution is:
[
  [2,4],
  [3,4],
  [2,3],
  [1,2],
  [1,3],
  [1,4],
]
```
利用通用模板,这是求2个的，若求所有的，时间O(2^n)
{% highlight C++ %}
vector<vector<int> > combine(int n, int k) {
  vector<vector<int> > res;
  if ((n == 0) || (k == 0)) return res;
  vector<int> path;
  combineCore(n, k, 1, res, path);  // start from 1 to n
  return res;
}
void combineCore(int n, int k, int start, vector<vector<int> > &res, vector<int> &path) {
  if (path.size() == k) {
    res.push_back(path);
    return;
  }
  for (int i = start; i <= n; i++) {
    path.push_back(i);
    combineCore(n, k, i + 1, res, path);
    path.pop_back();
  }
}
{% endhighlight %}
