---
layout: post
title:  "Basics on Dynamic Programming(DP) 8 - Max Matrix"
date:   2016-03-22 9:30:00
tags: [algorithm, leetcode, dynamic programming, dp, max matrix]
categories: Algorithm
---

> 一般来说处理矩阵的问题，大部分都是O(n^3)

### 1. 求n*n矩阵的最大矩阵（Leetcode最大和Maximum Subarray的升级版）
矩阵中的每个数可以正可以负, 极限复杂度也是O(n^3)
[参考解法](http://blog.csdn.net/beiyeqingteng/article/details/7056687)
{% highlight C++ %}
int maxSubsequence(int[] array) {
    if (array.length == 0) {
        return 0;
    }
    int max = Integer.MIN_VALUE;
    int[] maxSub = new int[array.length];
    maxSub[0] = array[0];
     
    for (int i = 1; i < array.length; i++) {
        maxSub[i] = (maxSub[i-1] > 0) ? (maxSub[i-1] + array[i]) : array[i]; 
        if (max < maxSub[i]) {
            max = maxSub[i];
        }
    }
    return max;
}
int subMaxMatrix(int[][] matrix) {  
    int[][] total = matrix;
    for (int i = 1; i < matrix[0].length; i++) {
        for (int j = 0; j < matrix.length; j++) {
            total[i][j] += total[i-1][j];
        }
    }
    int maximum = Integer.MIN_VALUE;
    for (int i = 0; i < matrix.length; i++) {
        for (int j = i; j < matrix.length; j++) {
            //result 保存的是从 i 行 到第 j 行 所对应的矩阵上下值的和
            int[] result = new int[matrix[0].length];
            for (int f = 0; f < matrix[0].length; f++) {
                if (i == 0) {
                    result[f] = total[j][f];
                } else {
                    result[f] = total[j][f] - total[i - 1][f];
                }
            }
            int maximal = maxSubsequence(result);
             
            if (maximal > maximum) {
                maximum = maximal;
            }
        }
    }
    return maximum;
}
{% endhighlight %}

### 2. [Maximal Rectangle - Leetcode 85 Hard](https://leetcode.com/problems/maximal-rectangle/)
```
Given a 2D binary matrix filled with 0's and 1's, find the largest rectangle containing all ones and return its area.
```
[O(n^2)解法:](http://blog.csdn.net/lanxu_yy/article/details/17533203)
{% highlight C++ %}
int maximalRectangle(vector<vector<char> > &matrix) {
    if (matrix.empty()) return 0;
    const int m = matrix.size();
    const int n = matrix[0].size();
    vector<int> H(n, 0), L(n, 0), R(n, n);
    int ret = 0;
    for (int i = 0; i < m; ++i) {
        int left = 0, right = n;
        // calculate L(i, j) from left to right
        for (int j = 0; j < n; ++j) {
            if (matrix[i][j] == '1') {
                ++H[j];
                L[j] = max(L[j], left);
            } else {
                left = j+1;
                H[j] = 0; L[j] = 0; R[j] = n;
            }
        }
        // calculate R(i, j) from right to left
        for (int j = n-1; j >= 0; --j) {
            if (matrix[i][j] == '1') {
                R[j] = min(R[j], right);
                ret = max(ret, H[j]*(R[j]-L[j]));
            } else {
                right = j;
            }
        }
    }
    return ret;
}
{% endhighlight %}
还有一种变体：问最大的对角线是1，其他部分是0的矩阵。
