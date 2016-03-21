---
layout: post
title:  "Basics on Dynamic Programming(DP) 2 - Matrix DP"
date:   2016-03-20 11:30:00
tags: [algorithm, leetcode, dynamic programming, dp]
categories: Algorithm
---

> 第1类常见DP问题：Matrix DP

### 1. [Unique Paths - Leetcode 62](https://leetcode.com/problems/unique-paths/)

```
A robot is located at the top-left corner of a m x n grid (marked 'Start' in the diagram below).
The robot can only move either down or right at any point in time. The robot is trying to reach the bottom-right corner of the grid (marked 'Finish' in the diagram below).
How many possible unique paths are there?
```

DP思路:

*  state: f[i][j] 代表了，从起点到达i,j这个点的路径方案之和 
*  function: f[i][j] = f[i - 1][j] + f[i][j - 1] 
*  intialize: f[i][0] 和 f[0][i] = 1, f[1][1] = 1 二维的矩阵的这种题目，一般初始化两条边
*  answer: f[n][m]

二维代码：
{% highlight C++ %}
int uniquePaths(int m, int n) {
    vector<vector<int> > dp(m, vector<int>(n, 0));  //全0的二维数组
    //intialize
    for(int i=0; i<m; i++)
        dp[i][0] = 1;
    for(int i=0; i<n; i++)
        dp[0][i] = 1;
    for(int i=1; i<m; i++) {
        for(int j=1; j<n; j++)
            dp[i][j] = dp[i-1][j] + dp[i][j-1];
    }
    return dp[m-1][n-1];
}
{% endhighlight %}
压缩一维：
{% highlight C++ %}
int uniquePaths(int m, int n) {
    vector<int> dp(n, 1);  //全0的数组,只跟左，上有关系，所以可以压缩.都初始化为1
    //intialize 已初始化
    for(int i=1; i<m; i++) {
        for(int j=1; j<n; j++)
            dp[j] = dp[j] + dp[j-1];
    }
    return dp[n-1];
}
{% endhighlight %}

### 2. [Unique Paths II - Leetcode 63](https://leetcode.com/problems/unique-paths-ii/)
```
Follow up for "Unique Paths":

Now consider if some obstacles are added to the grids. How many unique paths would there be?
An obstacle and empty space is marked as 1 and 0 respectively in the grid.

For example,
There is one obstacle in the middle of a 3x3 grid as illustrated below.

[
  [0,0,0],
  [0,1,0],
  [0,0,0]
]

The total number of unique paths is 2.
Note: m and n will be at most 100.
```
直接来个压缩算法：
{% highlight C++ %}
int uniquePathsWithObstacles(vector<vector<int> > &obstacleGrid) {
    int m = obstacleGrid.size();
    int n = obstacleGrid[0].size();
    vector<int> dp(n, 0);  //都初始化为0,有的是1障碍，需处理
    //intialize
    for(int i=0; i<n; i++) {
        if(obstacleGrid[0][i] == 1)
            break;  //从这儿往后都是0了，不用赋值了
        dp[i] = 1;
    }
    for(int i=1; i<m; i++) {
        for(int j=0; j<n; j++)  //这里跟上题有区别，因为有障碍了，不能从第2个开始!
            if(obstacleGrid[i][j] == 1)
                dp[j] = 0;
            else
                dp[j] = dp[j] + (j==0 ? 0 : dp[j-1]);   //从第1个开始的话，注意越界!!
    }
    return dp[n-1];
}
{% endhighlight %}

### 3. [Minimum Path Sum - Leetcode 64](https://leetcode.com/problems/minimum-path-sum/)
```
Given a m x n grid filled with non-negative numbers, find a path from top left to bottom right which minimizes the sum of all numbers along its path.
Note: You can only move either down or right at any point in time.
```
* state: f[i][j] 代表了，从起点到达i,j这个点的最短路
* function: f[i][j] = min(f[i - 1][j], f[i][j - 1]) + a[i][j];
* intialize: f[i][0] 和 f[0][i] = MAXINT(围墙，堵死，只能从1,1走), f[1][1] = a[0][0]
* answer: f[n][m]

直接压缩：
{% highlight C++ %}
int minPathSum(vector<vector<int> > &grid) {
    int m = grid.size();
    if(m == 0)  return 0;
    int n = grid[0].size();
    if(n == 0)  return 0;
     
    vector<int> dp(n);
    //initialize
    dp[0] = grid[0][0];
    for(int i=1; i<n; i++)
        dp[i] = dp[i-1] + grid[0][i];
     
    for(int i=1; i<m; i++) {
        dp[0] += grid[i][0];
        for(int j=1; j<n; j++)
            dp[j] = min(dp[j-1], dp[j]) + grid[i][j];
    }
    return dp[n-1];
}
{% endhighlight %}
