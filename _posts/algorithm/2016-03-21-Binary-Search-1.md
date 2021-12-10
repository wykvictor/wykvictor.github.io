---
layout: post
title:  "Binary Search Related - 1 Template"
date:   2016-03-21 13:30:00
tags: [algorithm, leetcode, binary search]
categories: Algorithm
---

### 1. Level 1 - Knows the template - 4 tips
一般O(n)肯定能解决的暴力题目，都会有O(logn)的解法，那就是二分
* start + 1 < end
* start + (end - start) / 2
* A[mid] ==, <, >
* A[start] A[end] ? target

### 2. Binary Search Template
Last Position of Target:    
{% highlight C++ %}
int lastPosition(vector<int>& A, int target) {  // A若是new，则在heap空间，A的指针在stack
  if (A.empty()) return -1;
  int start = 0, end = A.size() - 1;
  // 思想：while循环，负责缩小区间范围
  while (start + 1 < end) {
    int mid = start + (end - start) / 2;
    if (A[mid] == target) {
      start = mid;  // 模板：这句区别(start/end: 不要丢解)
    } else if (A[mid] < target) {
      start = mid;  // 写完后考虑，上边这2个if可以合并成1个
    } else {
      // 写作 end = mid - 1 也是正确的
      // 只是可以偷懒不写，因为不写也没问题，不会影响时间复杂度
      end = mid;
    }
  }
  // 出来后，负责肉眼可解的范围
  if (A[end] == target) return end;  //模板：这2句顺序区别
  if (A[start] == target) return start;
  return -1;
}
{% endhighlight %}
