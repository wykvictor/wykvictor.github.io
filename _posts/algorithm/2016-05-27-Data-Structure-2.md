---
layout: post
title:  "Data Structure 2 - Heap/TreeMap"
date:   2016-05-27 18:30:00
tags: [algorithm, leetcode, data structure, heap]
categories: Algorithm
---

### 1. [Heapify](http://www.lintcode.com/en/problem/heapify/)
```
Given an integer array, heapify it into a min-heap array.
For a heap array A, A[0] is the root of heap, and for each A[i], A[i * 2 + 1] is the left child of A[i] and A[i * 2 + 2] is the right child of A[i].

Given [3,2,1,4,5], return [1,2,3,4,5] or any legal heap array.
O(n) time complexity
```
{% highlight C++ %}
// Time: O(n)
void shiftDown(vector<int> &A, int cur) {
  int N = A.size();
  while (cur < N) {
    if (cur * 2 + 1 >= N) return;  // no child
    int replace;
    if (cur * 2 + 2 >= N) {  // only left child
      replace = cur * 2 + 1;
    } else {
      replace = A[cur * 2 + 2] > A[cur * 2 + 1] ? cur * 2 + 1 : cur * 2 + 2;
    }
    if (A[cur] <= A[replace]) return;
    swap(A[cur], A[replace]);
    cur = replace;
  }
}

void heapify(vector<int> &A) {
  int N = A.size();
  for (int i = N / 2; i >= 0; i--) {
    shiftDown(A, i);
  }
}
{% endhighlight %}

### 2. [Ugly Number II](http://www.lintcode.com/en/problem/heapify/)
```
Ugly number is a number that only have factors 2, 3 and 5.
Design an algorithm to find the nth ugly number. The first 10 ugly numbers are 1, 2, 3, 4, 5, 6, 8, 9, 10, 12...
If n=9, return 10.
Challenge: O(nlogn) or O(n) time
```
{% highlight C++ %}
// 1, { 乘2, 3, 5 }
// 2, 3, 5 4, 6, 10 { 每次拿出堆里最小的元素，乘以2, 3, 5 }
// 需要很快的add, min == > pq 另：2 * 5, 5 * 2 = 10，用hash记录是否10已经放过了
int nthUglyNumber(int n) {

}
{% endhighlight %}
