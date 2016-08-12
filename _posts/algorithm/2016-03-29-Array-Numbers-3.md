---
layout: post
title:  "Array & Numbers 3 - Sorted Array"
date:   2016-03-29 10:30:00
tags: [algorithm, leetcode, array, sorted array]
categories: Algorithm
---

### 1. [Merge Sorted Array](http://www.lintcode.com/en/problem/merge-sorted-array/)
```
Given two sorted integer arrays A and B, merge B into A as one sorted array.
```
{% highlight C++ %}
void mergeSortedArray(int A[], int m, int B[], int n) {
  // 倒着来，防止覆盖!
  int index = m + n - 1, i = m - 1, j = n - 1;
  while (i >= 0 && j >= 0) {
    A[index--] = A[i] > B[j] ? A[i--] : B[j--];
  }
  /* ! 不需要再弄A里的了，B完了，那么A就是原来的A 
  while (i >= 0) {
    A[index--] = A[i--];
  } */
  while (j >= 0) {
    A[index--] = B[j--];
  }
}
{% endhighlight %}

### 2. [Merge k Sorted Arrays](http://www.lintcode.com/en/problem/merge-k-sorted-arrays/)
```
Do it in O(N log k).
N is the total number of integers.
k is the number of arrays.
```

priority_queue的使用
{% highlight C++ %}
struct Node {
  int val;
  int x, y;  // 坐标
  Node(int a, int b, int c) : val(a), x(b), y(c) {}
};

struct cmp {
  bool operator()(Node a, Node b) { return a.val > b.val; }
};

vector<int> mergekSortedArrays(vector<vector<int>>& arrays) {
  vector<int> res;
  if (arrays.empty()) return res;
  priority_queue<Node, vector<Node>, cmp> q;
  for (int i = 0; i < arrays.size(); i++) {
    if (!arrays[i].empty()) {
      q.push(Node(arrays[i][0], i, 0));
    }
  }
  while (!q.empty()) {
    Node tmp = q.top();
    q.pop();
    res.push_back(tmp.val);
    if (tmp.y + 1 < arrays[tmp.x].size()) {  // !! +1, 判断放不放下一个
      q.push(Node(arrays[tmp.x][tmp.y + 1], tmp.x, tmp.y + 1));
    }
  }
  return res;
}
{% endhighlight %}

### 3. [Intersection of Two Arrays](http://www.lintcode.com/en/problem/intersection-of-two-arrays/)
```
Given two arrays, write a function to compute their intersection.
```

sort & merge:
{% highlight C++ %}
vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
  vector<int> res;
  sort(nums1.begin(), nums1.end());
  sort(nums2.begin(), nums2.end());
  int i = 0, j = 0;
  while (i < nums1.size() && j < nums2.size()) {
    if (nums1[i] == nums2[j]) {
      if (res.empty() || nums1[i] != res[res.size() - 1])
        res.push_back(nums1[i]);
      i++;
      j++;
    } else if (nums1[i] < nums2[j]) {
      i++;
    } else {
      j++;
    }
  }
  return res;
}
{% endhighlight %}
方法2，sort & binary search: nums1排序，nums2依次二分查找
{% highlight C++ %}
// 方法3，用hash，unordered_set!!!额外空间
vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
  vector<int> res;
  unordered_set<int> hash1, hashres;
  for (auto i : nums1) {  // auto!!
    hash1.insert(i);  // 自动去除重复元素
  }
  for (auto i : nums2) {
    if (hash1.count(i) && !hashres.count(i)) {
      hashres.insert(i);
    }
  }
  for (auto i : hashres) {
    res.push_back(i);
  }
  return res;
}
{% endhighlight %}

### 4. [Kth Largest Element](http://www.lintcode.com/en/problem/kth-largest-element/)
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

### 5. [Median of two Sorted Arrays - Important!](http://www.lintcode.com/en/problem/median-of-two-sorted-arrays/)
```
Given A=[1,2,3,4,5,6] and B=[2,3,4,5], the median is 3.5.
Given A=[1,2,3] and B=[4,5], the median is 3.
The overall run time complexity should be O(log (m+n)).
```

[solution](http://www.cnblogs.com/yuzhangcmu/p/4138184.html)
{% highlight C++ %}
// 注意下标：K的含义，第k个，从1开始
double findMedianSortedArrays(vector<int> A, vector<int> B) {
  int len = A.size() + B.size();
  if (len % 2 == 1) {
    return findKthNum(A, 0, B, 0, len / 2 + 1);  // 注意：kth从1开始, 0不好想
  } else {
    return (findKthNum(A, 0, B, 0, len / 2) +
            findKthNum(A, 0, B, 0, len / 2 + 1)) / 2.0;  // 注意double, 2.0!
  }
}
int findKthNum(vector<int> &A, int startA, vector<int> B, int startB, int k) {
  // 返回条件，A、B为空，或找第0th数
  if (startA >= A.size()) {
    return B[startB + k - 1];
  }
  if (startB >= B.size()) {
    return A[startA + k - 1];
  }
  if (k == 1) {
    return A[startA] < B[startB] ? A[startA] : B[startB];
  }
  // 开始divide
  if (startA + k / 2 - 1 >= A.size()) {  // 1. A不够k/2长度，则扔掉B
    return findKthNum(A, startA, B, startB + k / 2, k - k / 2);  // k不-1
  }
  if (startB + k / 2 - 1 >= B.size()) {  // B同理
    return findKthNum(A, startA + k / 2, B, startB, k - k / 2);
  }
  // 判断越界后，比较midK
  int midKA = A[startA + k / 2 - 1], midKB = B[startB + k / 2 - 1];
  if (midKA < midKB) {  // 2. A的midK小于B,丢掉A，如1,2,3  4,5,6
    return findKthNum(A, startA + k / 2, B, startB, k - k / 2);
  } else {  // B同理
    return findKthNum(A, startA, B, startB + k / 2, k - k / 2);
  }
}
{% endhighlight %}