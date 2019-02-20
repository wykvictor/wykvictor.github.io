---
layout: post
title:  "Combination and Permutation 3 - Permutations"
date:   2016-03-23 9:30:00
tags: [algorithm, leetcode, DFS, permutation]
categories: Algorithm
---

### 1. [Permutations - Leetcode 46](https://leetcode.com/problems/Permutations/)
```
Given a collection of numbers, return all possible permutations.
For example,
[1,2,3] have the following permutations:
[1,2,3], [1,3,2], [2,1,3], [2,3,1], [3,1,2], and [3,2,1].
```

通用模板：与combination区别，需标记数组（或每次push前，find下path里是否已经有），同时不需start参数，Core中循环每次从0开始，推荐方法
{% highlight C++ %}
时间复杂度O(n!)，空间复杂度O(n)
vector<vector<int> > permute(vector<int> &num) {
  vector<vector<int> > res;
  vector<int> path;
  vector<bool> visited(num.size(), false);  //与组合区别
  permuteCore(res, path, num, visited);
  return res;
}
void permuteCore(vector<vector<int> > &res, vector<int> &path, vector<int> &num,
                 vector<bool> &visited) {
  if (path.size() == num.size()) {  // all numbers
    res.push_back(path);
    return;
  }
  for (int i = 0; i < num.size(); i++) {  // start from 0
    if (visited[i]) continue;             //访问过的，不能再访问了
    path.push_back(num[i]);
    visited[i] = true;
    permuteCore(res, path, num, visited);
    path.pop_back();
    visited[i] = false;
  }
}
{% endhighlight %}
其他方法：
a. swap方法
{% highlight C++ %}
vector<vector<int> > permute(vector<int> &num) {
  vector<vector<int> > res;
  permuteCore(res, num, 0);  // num就相当于path了
  return res;
}
void permuteCore(vector<vector<int> > &res, vector<int> &num, int start) {
  if (start == num.size()) {
    res.push_back(num);
    return;
  }
  // 递归：整体分两部分，1是所有可能出现在第一位的(通过交换得到)，2是剩下的 继续递归做
  for (int i = start; i < num.size(); i++) {  //注意：i=start
    // 交换数组的元素，在递归后需要交换回来
    // 注意循环前要首先递归一次!!!本身原来的顺序就是第一种情况
    swap(num[start], num[i]);
    permuteCore(res, num, start + 1);  //换了后，求start后边开始的结果
    swap(num[start], num[i]);   
  }
}
{% endhighlight %}
b. STL法
{% highlight C++ %}
vector<vector<int> > permute(vector<int> &num) {
  //利用STL中std::next_permutation()，时间复杂度O(n!)，空间复杂度 O(1)
  vector<vector<int> > res;
  sort(num.begin(), num.end());
  do {
    res.push_back(num);
  } while (next_permutation(num.begin(), num.end()));
  return res;
}
{% endhighlight %}
c. 插入法,迭代：推荐方法
{% highlight C++ %}
vector<vector<int> > permute(vector<int> &num) {
  //插入法，迭代
  vector<vector<int> > res(1);            // need one 初始值
  for (int i = 0; i < num.size(); i++) {  // for each num, insert it into...
    //这种只要最后的结果的，都需要copyRes来赋值一份，跟combination相同
    vector<vector<int> > copyRes(res);
    res.clear();
    for (int j = 0; j < copyRes.size(); j++) {  // for each res, insert the i
      // there are res[j].size()+1 places to insert
      for (int k = 0; k <= copyRes[j].size(); k++) {
        vector<int> line(copyRes[j]);
        line.insert(line.begin() + k, num[i]);
        res.push_back(line);
      }
    }
  }
  return res;
}
{% endhighlight %}

### 2. [Permutations II - Leetcode 47](https://leetcode.com/problems/Permutations-ii/)
```
Given a collection of numbers that might contain duplicates, return all possible unique permutations.
For example,
[1,1,2] have the following unique permutations:
[1,1,2], [1,2,1], and [2,1,1]
```

通用模板：区别就是排序，且有continue判断
{% highlight C++ %}
vector<vector<int> > permuteUnique(vector<int> &num) {
  vector<vector<int> > res;
  vector<int> path;
  vector<bool> visited(num.size(), false);
  sort(num.begin(), num.end());  //这里需要排序!
  permuteCore(res, path, num, visited);
  return res;
}
void permuteCore(vector<vector<int> > &res, vector<int> &path, vector<int> &num,
                 vector<bool> &visited) {
  if (path.size() == num.size()) {  // all numbers
    res.push_back(path);
    return;
  }
  for (int i = 0; i < num.size(); i++) {
    if (visited[i] || (i != 0 && num[i] == num[i - 1] && visited[i - 1]))
      //访问过的，和上一个相同的 且上一个访问过的，都不能再访问了
      //只有这儿不同； 如1,1这样的，只有第2个1进入，然后第一个1才能入，顺序的
      continue;
    path.push_back(num[i]);
    visited[i] = true;
    permuteCore(res, path, num, visited);
    path.pop_back();
    visited[i] = false;  
  }
}
{% endhighlight %}
其他方法：迭代+判重
{% highlight C++ %}
vector<vector<int> > permuteUnique(vector<int> &num) {
  //插入法，迭代 ==> 判断重复
  vector<vector<int> > res(1);            // need one初始值
  for (int i = 0; i < num.size(); i++) {  // for each num, insert it into...
    int resNum = res.size();
    vector<vector<int> > copyRes(res);
    res.clear();
    for (int j = 0; j < resNum; j++) {  // for each res, insert the i
      for (int k = 0; k <= copyRes[j].size(); k++) {
        vector<int> line(copyRes[j]);
        line.insert(line.begin() + k, num[i]);
        // 就算排序了，也得有!
        if (find(res.begin(), res.end(), line) == res.end())
          res.push_back(line);
        //过滤重复的位置==>必须有这句优化，否则超时！
        while (k < line.size() - 1 && num[i] == line[k + 1]) k++;
      }
    }
  }
  return res;
}
{% endhighlight %}

### 3. [Next Permutation - Leetcode 31](https://leetcode.com/problems/next-permutation/)
```
Implement next permutation, which rearranges numbers into the lexicographically next greater permutation of numbers.
If such arrangement is not possible, it must rearrange it as the lowest possible order (ie, sorted in ascending order).
The replacement must be in-place, do not allocate extra memory.
Here are some examples. Inputs are in the left-hand column and its corresponding outputs are in the right-hand column.
1,2,3 → 1,3,2
3,2,1 → 1,2,3
1,1,5 → 1,5,1
```
![nextpermutation](/res/nextpermutation.png)
{% highlight C++ %}
void nextPermutation(vector<int> &num) {
  int index1 = num.size() - 1, index2 = num.size() - 1;
  while (index1 > 0 && num[index1] <= num[index1 - 1]) {
    index1--;
  }
  if (index1 > 0) {
    // 注意相同的情况1,1,5-> 5,1,1
    while (index2 >= index1 && num[index2] <= num[index1 - 1]) {
      index2--;
    }
    swap(num[index1 - 1], num[index2]);
  }
  reverse(num.begin() + index1, num.end());
}
{% endhighlight %}

### 4. [Permutation Sequence - Leetcode 60](https://leetcode.com/problems/permutation-sequence/)
```
The set [1,2,3,…,n] contains a total of n! unique permutations.

By listing and labeling all of the permutations in order,
We get the following sequence (ie, for n = 3):
"123"
"132"
"213"
"231"
"312"
"321"
Given n and k, return the kth permutation sequence.
Note: Given n will be between 1 and 9 inclusive.
```

通用模板会超时，找规律并计算
{% highlight Java %}
public String getPermutation(int n, int k) {
  ArrayList<Integer> nums = new ArrayList<Integer>();
  int factorial = 1;
  for (int i = 1; i <= n; i++) {
    nums.add(i);
    factorial = factorial * i;  // n!
  }
  String result = "";
  k--;  // Don`t foget, begin from 1th!
  for (int i = 0; i < n; i++) {
    factorial /= n - i;
    int cur = k / factorial;
    k = k % factorial;
    result += nums.get(cur);
    nums.remove(cur);
  }
  return result;
}
{% endhighlight %}
