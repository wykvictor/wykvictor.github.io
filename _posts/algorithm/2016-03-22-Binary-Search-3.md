---
layout: post
title:  "Binary Search Related - 3"
date:   2016-03-22 13:40:00
tags: [algorithm, leetcode, binary search]
categories: Algorithm
---

> Level 3 - Keep Half which Contain Results

### 1. [Find Peak Element - Leetcode 162 - 边界Hard](https://leetcode.com/problems/find-peak-element/)
```
A peak element is an element that is greater than its neighbors.
Given an input array where num[i] ≠ num[i+1], find a peak element and return its index.
The array may contain multiple peaks, in that case return the index to any one of the peaks is fine.

You may imagine that num[-1] = num[n] = -∞.
For example, in array [1, 2, 3, 1], 3 is a peak element and your function should return the index number 2.

Note: Your solution should be in logarithmic complexity.
```

Template solution:
{% highlight C++ %}
int findPeak(vector<int> A) {
  int beg = 0, end = A.size() - 1;
  while(beg + 1 < end) {
    int mid = beg + (end-beg)/2;
    // 4 types of positions:
    if(A[mid] > A[mid-1] && A[mid] > A[mid+1]) {
      return mid;
    } else if(A[mid] > A[mid-1] && A[mid] < A[mid+1]) {
      beg = mid;
    } else { // merge into 1 if
      end = mid;
    }
  }
  if(A[beg] > A[end])  return beg;
  return end;
}
//简化写法：
int findPeak(vector<int> A) {
  int beg = 0, end = A.size() - 1;
  while (beg + 1 < end) {
    int mid = beg + (end - beg) / 2;
    // 也对，但是没有上边运行返回快
    if (A[mid] > A[mid - 1]) {
      beg = mid;
    } else {  // merge into 1 if
      end = mid;
    }
  }
  if (A[beg] > A[end]) return beg;
  return end;
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

### 2. [Search for a Range](http://www.lintcode.com/en/problem/search-for-a-range/)
```
Given a sorted array of n integers, find the starting and ending position of a given target value.
If the target is not found in the array, return [-1, -1]

Given [5, 7, 7, 8, 8, 10] and target value 8,
return [3, 4].
```
{% highlight C++ %}
vector<int> searchRange(vector<int> &A, int target) {
  vector<int> res(2, -1);
  if(A.empty())  return res;
  // first position
  int beg = 0, end = A.size()-1;
  while(beg + 1 < end) {
    int mid = beg + (end-beg)/2;
    if(A[mid] >= target) {
      end = mid;
    } else {
      beg = mid;
    }
  }
  if(A[beg] == target) res[0] = beg;
  else if(A[end] == target) res[0] = end;
  else return res;
  // last position
  beg = 0, end = A.size()-1;
  while(beg + 1 < end) {
    int mid = beg + (end-beg)/2;
    if(A[mid] <= target) {
      beg = mid;
    } else {
      end = mid;
    }
  }
  if(A[end] == target) res[1] = end;
  else if(A[beg] == target) res[1] = beg;
  else return res;
}
{% endhighlight %}

### 3. [Count of Smaller Number](http://www.lintcode.com/en/problem/count-of-smaller-number/#)
```
Give you an integer array (index from 0 to n-1, where n is the size of this array, value from 0 to 10000) and an query list. For each query, give you an integer, return the number of element in the array that are smaller than the given integer.
```

Sort and binary search:
{% highlight C++ %}
// find first position >= target
// better than: last position < target, need to +1 in the end
int binarySearch(vector<int> &A, int target) {
  if(A.empty())  return 0;
  int beg = 0, end = A.size()-1;
  while(beg + 1 < end) {
    int mid = beg + (end-beg)/2;
    if(A[mid] < target) { // take care
      beg = mid;
    } else {
      end = mid;
    }
  }
  if(A[beg] >= target)  return beg;
  if(A[end] >= target)  return end;
  return 0;  // if not exist
}
vector<int> countOfSmallerNumber(vector<int> &A, vector<int> &queries) {
  vector<int> res;
  if(queries.empty())  return res;  // ask to make sure if they are empty!
  sort(A.begin(), A.end());
  for(int i=0; i<queries.size(); i++) {
    res.push_back(binarySearch(A, queries[i]));
  }
  return res;
}
{% endhighlight %}
Build Segment Tree and Search: TODO

### 4. [Closest Number in Sorted Array](http://www.lintcode.com/en/problem/closest-number-in-sorted-array/)
```
Given a target number and an integer array A sorted in ascending order, find the index i in A such that A[i] is closest to the given target.

Return -1 if there is no element in the array.
There can be duplicate elements in the array, and we can return any of the indices with same value.

Given [1, 4, 6] and target = 3, return 1.
Given [1, 4, 6] and target = 5, return 1 or 2.
```
{% highlight C++ %}
int closestNumber(vector<int>& A, int target) {
  if(A.empty())  return -1;
  int beg = 0, end = A.size()-1;
  // find 2 closest number
  while(beg + 1 < end) {
    int mid = beg + (end-beg)/2;
    if(A[mid] == target) {
      return mid;
    } else if(A[mid] < target) {
      beg = mid;
    } else {
      end = mid;
    }
  }
  // compare these 2
  if(abs(A[beg] - target) < abs(A[end] - target))
    return beg;
  return end;
}
{% endhighlight %}

### 5. [K Closest Numbers In Sorted Array](http://www.lintcode.com/en/problem/k-closest-numbers-in-sorted-array/)
```
Given a target number, a non-negative integer k and an integer array A sorted in ascending order, find the k closest numbers to target in A, sorted in ascending order by the difference between the number and target. Otherwise, sorted in ascending order by number if the difference is same.

Given A = [1, 2, 3], target = 2 and k = 3, return [2, 1, 3].
Given A = [1, 4, 6, 8], target = 3 and k = 3, return [4, 1, 6].

O(logn + k) time complexity.
```
{% highlight C++ %}
int lastSmaller(vector<int>& A, int target) {
  int beg = 0, end = A.size()-1;
  while(beg + 1 < end) {
    int mid = beg + (end-beg)/2;
    if(A[mid] < target) {
      beg = mid;
    } else {
      end = mid;
    }
  }
  if(A[end] < target)  return end;
  return beg;
}

vector<int> kClosestNumbers(vector<int>& A, int target, int k) {
  vector<int> res;
  if(A.empty() || A.size() < k || k <= 0)  return res;
  // find the mid-index to start extending
  int mid = lastSmaller(A, target);
  int left = mid, right = mid+1;
  while(left >= 0 && right < A.size()) {
    if(abs(A[left] - target) <= abs(A[right] - target)) {
      res.push_back(A[left]);
      left--;
    } else if(abs(A[left] - target) > abs(A[right] - target)){
      res.push_back(A[right]);
      right++;
    }
    if(res.size() == k)
      return res;
  }
  if(left >= 0) {
    for(int i = res.size(); i < k; i++) {
      res.push_back(A[left--]);
    }
  } else if(right < A.size()){
    for(int i = res.size(); i < k; i++) {
      res.push_back(A[right++]);
    }
  }
  return res;
}

// 简化逻辑：
vector<int> kClosestNumbers(vector<int>& A, int target, int k) {
  vector<int> res;
  if(A.empty() || A.size() < k || k <= 0)  return res;
  // find the mid-index to start extending
  int mid = lastSmaller(A, target);
  int left = mid, right = mid+1;
  // 前边边界判断已经保证，能找到k个
  for(int i = 0; i < k; i++) {
    if(left < 0) {
      res.push_back(A[right++]);
    } else if(right >= A.size()) {
      res.push_back(A[left--]);
    } else if(abs(A[left] - target) <= abs(A[right] - target)) {
      res.push_back(A[left--]);
    } else {
      res.push_back(A[right++]);
    }
  }
  return res;
}
{% endhighlight %}

### 6. [Smallest Rectangle Enclosing Black Pixels](http://www.lintcode.com/en/problem/smallest-rectangle-enclosing-black-pixels/)
```
An image is represented by a binary matrix with 0 as a white pixel and 1 as a black pixel. The black pixels are connected, i.e., there is only one black region. Pixels are connected horizontally and vertically. Given the location (x, y) of one of the black pixels, return the area of the smallest (axis-aligned) rectangle that encloses all black pixels.
For example, given the following image:
[
  "0010",
  "0110",
  "0100"
]
and x = 0, y = 2,
Return 6.
```
{% highlight C++ %}
// BFS, 时间复杂度O(n)
// Binary Search, O(logn)
int minArea(vector<vector<char>>& image, int x, int y) {
  int left = y, right = y, top = x, bottom = x;
  int rows = image.size();
  if (rows == 0) return 0;
  int cols = image[0].size();
  if (cols == 0) return 0;

  // search left
  int l = 0, r = left;
  while (l + 1 < r) {
    int mid = l + (r - l) / 2;
    if (searchCols(image, mid)) {  // if has 1
      r = mid;
    } else {
      l = mid;
    }
  }
  left = searchCols(image, l) ? l : r;
  // search right
  l = right;
  r = cols - 1;
  while (l + 1 < r) {
    int mid = l + (r - l) / 2;
    if (searchCols(image, mid)) {  // if has 1
      l = mid;
    } else {
      r = mid;
    }
  }
  right = searchCols(image, r) ? r : l;
  // search top
  l = 0, r = top;
  while (l + 1 < r) {
    int mid = l + (r - l) / 2;
    if (searchRows(image, mid)) {  // if has 1
      r = mid;
    } else {
      l = mid;
    }
  }
  top = searchRows(image, l) ? l : r;
  // search bottom
  l = bottom, r = rows - 1;
  while (l + 1 < r) {
    int mid = l + (r - l) / 2;
    if (searchRows(image, mid)) {  // if has 1
      l = mid;
    } else {
      r = mid;
    }
  }
  bottom = searchRows(image, r) ? r : l;

  return (right - left + 1) * (bottom - top + 1);
}

bool searchCols(vector<vector<char>>& image, int c) {
  for (int i = 0; i < image.size(); i++) {
    if (image[i][c] == '1') return true;
  }
  return false;
}

bool searchRows(vector<vector<char>>& image, int r) {
  for (int i = 0; i < image[0].size(); i++) {
    if (image[r][i] == '1') return true;
  }
  return false;
}
{% endhighlight %}
