---
layout: post
title:  "Data Structure 1 - Queue/Stack/Hash"
date:   2016-05-25 18:30:00
tags: [algorithm, leetcode, queue, stack, hash]
categories: Algorithm
---

### 1. [Min Stack](http://www.lintcode.com/en/problem/min-stack/)
```
The stack should support push, pop and min operation all in O(1) cost
```
{% highlight C++ %}
class MinStack {
 public:
  MinStack() {}

  void push(int number) {
    data.push(number);
    // 是否更新最小值, 注意等于号!
    if (mins.empty() || number <= mins.top()) mins.push(number);
  }

  int pop() {
    int cur = data.top();
    if (cur == mins.top()) {
      mins.pop();
    }
    data.pop();
    return cur;
  }

  int min() { return mins.top(); }

 private:
  stack<int> data;
  stack<int> mins;  // 记录最小值变化序列
};
{% endhighlight %}

### 2. [Largest Rectangle in Histogram](http://www.lintcode.com/en/problem/largest-rectangle-in-histogram/)
```
Given n non-negative integers representing the histogram's bar height where the width of each bar is 1, find the area of largest rectangle in the histogram.
Given height = [2,1,5,6,2,3], return 10
```

为何不能使用DP：是求最大；是必须有序序列；但不是暴力复杂度

此题暴力解法为O(n^2), DP不适合优化此类算法(适合2^n->n^2)

nlogn: merge sort, quick sort(两半，O(n)时间合起来); 先sort; for...内部数据结构logn的操作，Tree Style

n: 单调栈 [1,4,5] -> push(2) -> [1,2]

一个pop操作的时候，可以知道某个数左边第一个比它小的数；右边，第一个比它小的数

解法：木桶原理，求每个柱子往左往右最大延伸的矩形(左右第一个比它小的数)

![Histogram-Largest](http://7xno5y.com1.z0.glb.clouddn.com/Histogram-Largest.jpg)

{% highlight C++ %}
// O(n) time and memory, 每个数在stack上都进出一次；同理，binary tree相关，每个节点O(1),共n个点，所以O(n)
int largestRectangleArea(vector<int> &height) {
  int res = 0;  // 初始值为0，最小或为空都是0
  stack<int> incStack;
  for (int i = 0; i <= height.size(); i++) {
    int cur = i == height.size() ? -1 : height[i];  // 最后填-1把所有的弹出来
    while (!incStack.empty() && cur < height[incStack.top()]) {  // cur <= 也可以!
      int h = incStack.top();
      incStack.pop();
      int w = incStack.empty() ? i : i - incStack.top() - 1;  // 空的时候为i
      res = max(res, height[h] * w);
    }
    incStack.push(i);
  }
  return res;
}
{% endhighlight %}

### 3. [Maximal Rectangle](http://www.lintcode.com/en/problem/maximal-rectangle/)
```
Given a 2D boolean matrix filled with False and True, find the largest rectangle containing all True and return its area.
Given a matrix:
[
  [1, 1, 0, 0, 1],
  [0, 1, 0, 0, 1],
  [0, 0, 1, 1, 1],
  [0, 0, 1, 1, 1],
  [0, 0, 0, 0, 1]
]
return 6.
```
{% highlight C++ %}
int maximalRectangle(vector<vector<bool> > &matrix) {
  // O(n^4) 暴力解法, TLE超时
  int res = 0;
  if (matrix.size() == 0) return res;
  for (int i = 0; i < matrix.size(); i++) {
    for (int j = 0; j < matrix[0].size(); j++) {
      // 枚举每个左上角，之后枚举右下角
      if (matrix[i][j] == 0) continue;
      int maxy = matrix[0].size() - 1;  // 右边最远的位置
      for (int k = i; k < matrix.size(); k++) {
        for (int z = j; z <= maxy; z++) {
          if (matrix[k][z] == 0) {
            maxy = z - 1;
            break;
          }
          res = max(res, (k - i + 1) * (z - j + 1));
        }
      }
    }
  }
  return res;
}
{% endhighlight %}
单调栈解法O(n^2)
{% highlight C++ %}
int maximalRectangle(vector<vector<bool> > &matrix) {
  int rows = matrix.size();
  if (rows == 0) return 0;
  int cols = matrix[0].size();
  vector<int> height(cols, 0);
  int res = 0;
  for (int i = 0; i < rows; i++) {
    for (int j = 0; j < cols; j++) {
      if (matrix[i][j] == 1) {
        height[j]++;
      } else {
        height[j] = 0;
      }
    }
    res = max(res, largestRectangleArea(height)); // 上题函数
  }
  return res;
}
{% endhighlight %}

### 4. [Max Tree](http://www.lintcode.com/en/problem/max-tree/)
```
Given an integer array with no duplicates. A max tree building on this array is defined as follow:

The root is the maximum number in the array
The left subtree and right subtree are the max trees of the subarray divided by the root number.

Construct the max tree by the given array.

Given [2, 5, 6, 0, 3, 1], the max tree constructed by this array is:
6,5,3,2,#,0,1
```
{% highlight C++ %}
// O(n) time and memory

{% endhighlight %}
