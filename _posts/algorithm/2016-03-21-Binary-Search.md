---
layout: post
title:  "Binary Search Related"
date:   2016-03-21 13:30:00
tags: [algorithm, leetcode, binary search]
categories: Algorithm
---

> mid = left + ((right-left)>>1);  // 移位快；防止移除；运算符优先级>>is lower than +/-

> 注意区间问题: 用mid赋值是否步进，要考虑死循环和提前跳出

### 1. [Find Peak Element - Leetcode 162 - 边界Hard](https://leetcode.com/problems/find-peak-element/)
```
A peak element is an element that is greater than its neighbors.
Given an input array where num[i] ≠ num[i+1], find a peak element and return its index.
The array may contain multiple peaks, in that case return the index to any one of the peaks is fine.

You may imagine that num[-1] = num[n] = -∞.
For example, in array [1, 2, 3, 1], 3 is a peak element and your function should return the index number 2.

Note: Your solution should be in logarithmic complexity.
```
{% highlight Java %}
public int findPeakElement(int[] nums) {
    int len = nums.length;
    if(len == 1) return 0;
    if(len == 2) return nums[0] < nums[1] ? 1 : 0;
    int i=0, j=len-1;
    while(i <= j) {   // 这样的话，下边是m+1,m-1
        if(i == j)  return i;
        int m = i + ((j-i)>>1);
        if(m == 0)  return nums[0] < nums[1] ? 1 : 0;  // 防止越界
        if(m == len-1)  return nums[m] > nums[m-1] ? len-1 : len-2;
        if(nums[m] > nums[m-1] && nums[m] > nums[m+1]) {  //这个不能少，防止m跑丢
        	return m;
        } else if(nums[m] > nums[m-1]) {
            i = m + 1;
        } else if(nums[m] > nums[m+1]) {
            j = m - 1;
        } else {  // 此时是个valley,往左右走都可以
        	j = m - 1;
    	}
    }
    return -1;
}
{% endhighlight %}
后2个else合成一个：
{% highlight Java %}
public int findPeakElement(int[] nums) {
    int len = nums.length;
    if(len == 1) return 0;
    if(len == 2) return nums[0] < nums[1] ? 1 : 0;
    int i=0, j=len-1;
    while(i <= j) {   // 这样的话，下边是m+1,m-1
        if(i == j)  return i;
        int m = i + ((j-i)>>1);
        if(m == 0)  return nums[0] < nums[1] ? 1 : 0;  // 防止越界
        if(m == len-1)  return nums[m] > nums[m-1] ? len-1 : len-2;
        if(nums[m] > nums[m-1] && nums[m] > nums[m+1]) {
        	return m;
        } else if(nums[m] > nums[m-1]) {
            i = m + 1;
        } else {  // 此时是个valley,往左右走都可以
        	j = m - 1;
    	}
    }
    return -1;
}
{% endhighlight %}
另一种，while(i < j)的步进方式:
{% highlight Java %}
public int findPeakElement(int[] nums) {
    if(nums.length == 1) return 0;
    int i=0, j=nums.length-1;
    while(i < j) {   // 这样的话，下边有1个步进
        int m = i + ((j-i)>>1);  // 这样是m与i接近，下包含
        if(nums[m] > nums[m+1]) {  //因此先比较m+1，否则m-1越界
            j = m;  // 这时，不会丢掉
        } else {
            i = m + 1;
        }
    }
    return i;
}
{% endhighlight %}

### 2. find first index
```
For a given sorted array (ascending order) and a target number, find the first index of this number in O(log n) time complexity.
If the target number does not exist in the array, return -1.

Example
If the array is [1, 2, 3, 3, 4, 5, 10], for given target 3, return 2.
```
{% highlight C++ %}
int binarySearch(vector<int> &array, int target) {
    // write your code here
    int begin=0, end=array.size()-1;
    which(begin <= end) {
        int mid = start + ((end - start) >> 1);
        if(A[mid] == target) {
            if(mid == 0 || A[mid-1] != target)
                return mid;
            end = mid - 1;
        } else if(A[mid] < target) {
            begin = mid + 1;
        } else {
            end = mid - 1;
        }
    }
    return -1;
}
{% endhighlight %}
