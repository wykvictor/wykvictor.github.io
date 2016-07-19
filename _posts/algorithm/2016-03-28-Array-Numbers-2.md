---
layout: post
title:  "Array & Numbers 1 - Two Pointers"
date:   2016-03-27 10:30:00
tags: [algorithm, leetcode, array, two pointer]
categories: Algorithm
---

### 1. [Kth Largest Element](http://www.lintcode.com/en/problem/kth-largest-element/)
```
Find K-th largest element in an array.
In array [9,3,2,4,8], the 3rd largest element is 4.
O(n) time, O(1) extra memory.
```
{% highlight C++ %}
int kthLargestElement(int k, vector<int> nums) {
  if (nums.size() == 0) return -1;
  return quickSelect(nums, nums.size() - k + 1, 0, nums.size() - 1);
}
// k is the k-th number in an increase array
int quickSelect(vector<int> &nums, int k, int start, int end) {
  if (start == end) {
    return nums[start];
  }
  int mid = start + (end - start) / 2;
  int pivot = nums[mid];
  // partition
  int i = start, j = end;
  while (i <= j) {
    while (i <= j && nums[i] < pivot) {
      i++;
    }
    while (i <= j && nums[j] > pivot) {
      j--;
    }
    if (i <= j) {
      swap(nums[i], nums[j]);
      i++;
      j--;
    }
  }  // i,j 错位了
  // 比较k和i，看在左右哪一堆
  if (start + k - 1 <= j) {
    return quickSelect(nums, k, start, j);
  }
  if (start + k - 1 >= i) {
    return quickSelect(nums, k - (i - start), i, end);
  }
  return nums[start + k - 1];
}
{% endhighlight %}
