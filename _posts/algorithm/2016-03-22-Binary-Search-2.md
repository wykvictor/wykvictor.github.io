---
layout: post
title:  "Binary Search Related - 2"
date:   2016-03-22 13:30:00
tags: [algorithm, leetcode, binary search]
categories: Algorithm
---

> Level 2 - Find first/last index/result under certain conditions

### Binary Search on Index:

### 1. [Find first index](http://www.lintcode.com/en/problem/first-position-of-target/)
```
For a given sorted array (ascending order) and a target number, find the first index of this number in O(log n) time complexity.
If the target number does not exist in the array, return -1.

Example
If the array is [1, 2, 3, 3, 4, 5, 10], for given target 3, return 2.
```
{% highlight C++ %}
// 旧解法：考虑很多边界条件，易错
int binarySearch(vector<int> &array, int target) {
  int begin=0, end=array.size()-1;
  while(begin <= end) {
    int mid = begin + ((end - begin) >> 1);
    if(array[mid] == target) {
      if(mid == 0 || array[mid-1] != target)
        return mid;
      end = mid - 1;
    } else if(array[mid] < target) {
      begin  = mid + 1;
    } else {
      end = mid - 1;
    }
  }
  return -1;
}
// 模板
int binarySearch(vector<int> &array, int target) {
  if(array.empty())  return -1;
  int begin=0, end=array.size()-1;
  while(begin + 1 < end) {
    int mid = begin + ((end - begin) >> 1);
    if(array[mid] == target) {
      end = mid;  // 不同
    } else if(array[mid] < target) {
      begin = mid;
    } else {
      end = mid;
    }
  }
  if(array[begin] == target)  return begin;  // 顺序变一下
  if(array[end] == target)    return end;
  return -1;
}
{% endhighlight %}

### 2. [Search Insert Position](http://www.lintcode.com/en/problem/search-insert-position/)
```
Given a sorted array and a target value, return the index if the target is found. If not, return the index where it would be if it were inserted in order.
You may assume *NO duplicates* in the array.
Example
[1,3,5,6], 5 → 2
[1,3,5,6], 2 → 1
[1,3,5,6], 0 → 0
```
{% highlight C++ %}
int searchInsert(vector<int> &A, int target) {
  if (A.empty()) return 0;  // Important: return 0 not -1!!
  int start = 0, end = A.size() - 1;
  // first position >=
  while (start + 1 < end) {
    int mid = start + (end - start) / 2;
    if (A[mid] == target) {
      end = mid;  //模板：这句区别(start/end: 不要丢解)
    } else if (A[mid] < target) {
      start = mid;
    } else {
      end = mid;
    }
  }
  if (A[start] >= target) return start;
  if (A[end] >= target) return end;  //模板：这2句顺序区别
  if (A[end] < target) return end + 1;  // 注意，这种特殊情况！
}
{% endhighlight %}

### 3. [Search in a Big Sorted Array](http://www.lintcode.com/en/problem/search-in-a-big-sorted-array/)
```
Given a big sorted array with positive integers sorted by ascending order. The array is so big so that you can not get the length of the whole array directly, and you can only access the kth number by ArrayReader.get(k) (or ArrayReader->get(k) for C++). Find the first index of a target number. Your algorithm should be in O(log k), where k is the first index of the target number.

Return -1, if the number doesn't exist in the array.
```
{% highlight C++ %}
int searchBigSortedArray(ArrayReader *reader, int target) {
  if(reader == NULL)  return -1;
  int begin = 0, end = 1;
  // 先找end
  while(reader->get(end) < target) {
      end = end * 2;
  }
  // 应该在end和end/2之间
  begin = end / 2;
  while(begin + 1 < end) {
      int mid = begin + (end-begin) / 2;
      if(reader->get(mid) == target) {
          end = mid;
      } else if (reader->get(mid) < target) {
          begin = mid + 1;
      } else {
          end = mid - 1;
      }
  }
  if(reader->get(begin) == target) return begin;
  if(reader->get(end) == target) return end;
  return -1;
}
{% endhighlight %}

### 4. [Find Minimum in Rotated Sorted Array](http://www.lintcode.com/en/problem/find-minimum-in-rotated-sorted-array/)
```
Suppose a sorted array is rotated at some pivot unknown to you beforehand.
(i.e., 0 1 2 4 5 6 7 might become 4 5 6 7 0 1 2).
Find the minimum element.
```
{% highlight C++ %}
int findMin(vector<int> &num) {
  int size = nums.size();
  if(size == 0)  return -1;
  int start = 0, end = size - 1;
  if(nums[start] < nums[end])  return nums[start];
  while(start + 1 < end) {
      int mid = start + (end - start) / 2;
      if(nums[mid] > nums[start]) {
          start = mid;
      } else if (nums[mid] < nums[start]) {
          end = mid;
      } else {
          break;
      }
  }
  return nums[end];
}
{% endhighlight %}

[Find Minimum in Rotated Sorted Array II](http://www.lintcode.com/en/problem/find-minimum-in-rotated-sorted-array-ii/)

The array may contain duplicates.
{% highlight C++ %}
// worst case: 1110111 (O(n), just use for loop)
int findMin(vector<int> &num) {
  int size = nums.size();
  if(size == 0)  return -1;
  int start = 0, end = size - 1;
  while(start + 1 < end) {
      if(nums[start] < nums[end])  return nums[start];  // 11110111，而且注意放到循环里！
      int mid = start + (end - start) / 2;
      if(nums[mid] > nums[start]) {
          start = mid;
      } else if (nums[mid] < nums[start]) {
          end = mid;
      } else if(nums[mid] == nums[start]) {
          // 无法判断mid在左/右区间，如11110111
          start++;
      } else if(nums[mid] == nums[end]) {
          // 无法判断mid在左/右区间，如11110111
          end--;
      }
  }
  if(nums[start] < nums[end])  return nums[start];
  return nums[end];
}
{% endhighlight %}

### 5. [Search in Rotated Sorted Array](http://www.lintcode.com/en/problem/search-in-rotated-sorted-array/)
```
If found in the array return its index, otherwise return -1.
You may assume no duplicate exists in the array.
```
{% highlight C++ %}
int search(vector<int> &A, int target) {
  // write your code here
  int size = A.size();
  if(size == 0)  return -1;
  int start = 0, end = size - 1;
  while(start + 1 < end) {
      int mid = start + (end - start) / 2;
      if(A[mid] == target)  return mid;
      // 分情况讨论，mid在左区间 or 右区间
      if(A[mid] > A[start]) {
          if(target < A[mid] && target >= A[start] ) { // ！必须是2个判断
              // 正常二分, A[start] <= target < A[mid]
              end = mid - 1;  // 这个外围while也支持正常的二分，都是进来A[mid] > A[start]
          } else {
              // 退化到初始状况，再来
              start = mid + 1;
          }
      } else {
          if(target > A[mid] && target <= A[end]) {
              // 正常二分, A[mid] < target <= A[end]
              start = mid + 1;
          } else {
              // 退化到初始状况，再来
              end = mid - 1;
          }
      }
  }
  if(A[start] == target)  return start;
  if(A[end] == target)  return end;
  return -1;
}
{% endhighlight %}
What if duplicates are allowed?
{% highlight C++ %}
bool search(vector<int> &A, int target) {
  // write your code here
  int size = A.size();
  if(size == 0)  return false;
  int start = 0, end = size - 1;
  while(start + 1 < end) {
      int mid = start + (end - start) / 2;
      if(A[mid] == target)  return true;
      // 分情况讨论，mid在左区间 or 右区间
      if(A[mid] > A[start]) {
          if(target < A[mid] && target >= A[start] ) { // 必须是2个判断
              // 正常二分, A[start] <= target < A[mid]
              end = mid - 1;  // 这个外围while也支持正常的二分，都是进来A[mid] > A[start]
          } else {
              // 退化到初始状况，再来
              start = mid + 1;
          }
      } else if(A[mid] < A[start]) {
          if(target > A[mid] && target <= A[end]) {
              // 正常二分, A[mid] < target <= A[end]
              start = mid + 1;
          } else {
              // 退化到初始状况，再来
              end = mid - 1;  // !上头判断过 A[mid] == target 了
          }
      } else {
          start++;
      }
  }
  if(A[start] == target)  return true;
  if(A[end] == target)  return true;
  return false;
}
{% endhighlight %}

### 6. [Search a 2D Matrix](https://www.lintcode.com/problem/28/)
```
写出一个高效的算法来搜索m × n矩阵中的值target
1. 每行中的整数从左到右是排序的。
2. 每行的第一个数大于上一行的最后一个整数。
```
{% highlight C++ %}
// 按照模版，2次二分O(logm) + O(logn)
bool searchMatrix(vector<vector<int>> &matrix, int target) {
  if(matrix.size() == 0) return false;
  if(matrix[0].size() == 0)  return false;
  int rows = matrix.size(), cols = matrix[0].size();
  // search which row, last row < target
  int begin = 0, end = rows - 1;
  while(begin + 1 < end) {
      int mid = begin + (end - begin) / 2;
      if(matrix[mid][0] == target) {
          return true;
      } else if (matrix[mid][0] < target) {
          begin = mid + 1;
      } else {
          end = mid - 1;
      }
  }
  int rowID = 0;
  if(matrix[end][0] <= target) {  // ！！重要，需要有“=”
      rowID = end;
  } else if(matrix[begin][0] <= target) {
      rowID = begin;
  } else {
      return false;
  }

  // search which col
  begin = 0, end = cols - 1;
  while(begin + 1 < end) {
      int mid = begin + (end-begin) / 2;
      int midval = matrix[rowID][mid];
      if(midval == target) {
          return true;
      } else if(midval < target) {
          begin = mid + 1;
      } else {
          end = mid - 1;
      }
  }
  if(matrix[rowID][begin] == target) return true;
  if(matrix[rowID][end] == target) return true;
  return false;
}
{% endhighlight %}
O(log(m*n))的方式，但是代码简洁
{% highlight C++ %}
bool searchMatrix(vector<vector<int>> &matrix, int target) {
  if(matrix.size() == 0) return false;
  if(matrix[0].size() == 0)  return false;
  int rows = matrix.size(), cols = matrix[0].size();
  int begin = 0, end = rows * cols - 1;
  while(begin + 1 < end) {
    int mid = begin + (end-begin) / 2;
    int midval = matrix[mid/cols][mid%cols];
    if(midval == target) {
        return true;
    } else if(midval < target) {
        begin = mid + 1;
    } else {
        end = mid - 1;
    }
  }
  if(matrix[begin/cols][begin%cols] == target) return true;
  if(matrix[end/cols][end%cols] == target) return true;
  return false;
}
{% endhighlight %}


### Binary Search on Result
往往没有给定数组让二分，同样是找满足某个条件的最大/小值

### 7. [First Bad Version - Leetcode 278](https://leetcode.com/problems/first-bad-version/)
```
You are a product manager and currently leading a team to develop a new product. Unfortunately, the latest version of your product fails the quality check. Since each version is developed based on the previous version, all the versions after a bad version are also bad.

Suppose you have n versions [1, 2, ..., n] and you want to find out the first bad one, which causes all the following ones to be bad.

You are given an API bool isBadVersion(version) which will return whether version is bad. Implement a function to find the first bad version. You should minimize the number of calls to the API.
```
{% highlight C++ %}
int firstBadVersion(int n) {
  int begin=1, end=n;
  while(begin + 1 < end) {
    int mid = begin + ((end - begin) >> 1);
    if(isBadVersion(mid)) {
      end = mid;
    } else {
      begin = mid;
    }
  }
  if(isBadVersion(begin))  return begin;
  return end;
}
{% endhighlight %}

### 8. [Sqrt(x)](http://www.lintcode.com/en/problem/sqrtx/)
{% highlight C++ %}
int sqrt(int x) {
  if(x <= 0)  return 0;
  // 问题转化：last number ^2 <= x
  long long beg = 1, end = x;  // long long is important!
  while(beg + 1 < end) {
    long long mid = beg + (end-beg)/2;  // 如果不用long，则需要写mid < x / mid，但除法时钟周期慢3倍
    long long mypow = mid * mid;  // mid必须long，否则要这么写(long long)
    if(mypow == x) {
      return mid;
    } else if (mypow < x) {
      beg = mid;  // 而且不能是mid + 1，会丢解
    } else {
      end = mid;
    }
  }
  if(end * end <= x)  return end;
  return beg;
}
{% endhighlight %}
若输入输出都是double，精确度保持在小数点后12位
{% highlight C++ %}
double sqrt(double x) {
    // write your code here
    if(x <= 0)  return 0;  // 特殊情况
    double eps = 1e-12;
    double start = 1, end = x;
    if(x < 1) {  // !特殊情况
        start = x;
        end = 1;
    }
    while(start + eps < end) {
        double mid = start + (end - start) / 2;
        if(abs(mid * mid - x) <= eps) {
            return mid;
        }
        if(mid * mid < x) {
            start = mid;
        } else {
            end = mid;
        }
    }
    return start;
}
{% endhighlight %}

### 9. [Wood Cut](https://leetcode.com/problems/first-bad-version/)
```
Given n pieces of wood with length L[i](integer array). Cut them into small pieces to guarantee you could have equal or more than k pieces with the same length. What is the longest length you can get from the n pieces of wood? Given L & k, return the maximum length of the small pieces.
For L=[232, 124, 456], k=7, return 114.
```
{% highlight C++ %}
// cut的长度，能否搞k个
bool getcut(vector<int> &L, int k, int cut) {
    int num = 0;
    for(auto l: L) {
        num += l/cut;
        if(num >= k) return true;
    }
    return false;
}

int woodCut(vector<int> &L, int k) {
    if(k == 0 || L.size() == 0) return 0;
    int maxlen = 1;
    for(auto l: L) {
        if(maxlen < l) maxlen = l;
    }
    int start = 1, end = maxlen;
    // last num to get k
    while(start + 1 < end) {
        int mid = start + (end - start) / 2;
        if(getcut(L, k, mid)) {
            start = mid;
        } else {
            end = mid - 1;
        }
    }
    if(getcut(L, k, end)) return end;
    if(getcut(L, k, start)) return start;
    return 0;
}
{% endhighlight %}
