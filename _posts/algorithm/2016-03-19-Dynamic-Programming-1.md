---
layout: post
title:  "Basics on Dynamic Programming(DP) 1 - Intro"
date:   2016-03-19 11:30:00
tags: [algorithm, leetcode, dynamic programming, dp]
categories: Algorithm
---

> Learn the basics on DP througe some examples

### 1. Why use DP - [Triangle - Leetcode 120](https://leetcode.com/problems/triangle/)
```
Given a triangle, find the minimum path sum from top to bottom. Each step you may move to adjacent numbers on the row below.

For example, given the following triangle
[
     [2],
    [3,4],
   [6,5,7],
  [4,1,8,3]
]
The minimum path sum from top to bottom is 11 (i.e., 2 + 3 + 5 + 1 = 11).

Note:
Bonus point if you are able to do this using only O(n) extra space, where n is the total number of rows in the triangle.
```
1) 原始的解法, DFS解法, 爆炸性复杂度2^n, 伪代码:
{% highlight C++ %}
// traversal 
int minimumTotal(vector<vector<int> > &triangle) {
  int n = triangle.size();  // n is the height
  int best = INT_MAX;
  dfs(triangle, 0, 0, 0, best);  // sum is root->x,y, not include root
  return best;
}
void dfs(vector<vector<int> > &triangle, int x, int y, int sum, int &best) {
  if(x == triangle.size()) {
    best = min(best, sum);
    return;
  }
  dfs(triangle, x + 1, y, sum + triangle[x][y], best);
  dfs(triangle, x + 1, y+1, sum + triangle[x][y], best); //sum这时包括了root点
}
{% endhighlight %}
2) 此类方法的优化：

* 如果没解了，就不搜了（可行性剪枝） 不可行
* 如果解不可能最优，就不搜了（最优性剪枝）  可尝试
* 看看有没有重复计算, 伪代码如下：
{% highlight C++ %}
// 代表了从xy出发，走到最底层的最短路是多少. 变化：有返回值了
int dfs(int x, int y) {    
    if (x == n) {
        return 0;
    }
    
    if (flag[x][y]) {  //重复计算。标记算过了
        return hash[x][y];
    }
    flag[x][y] = true;
    hash[x][y] = min(dfs(x + 1, y), dfs(x + 1, y + 1)) + a[x][y];
    return hash[x][y];
}
 
dfs(0, 0)
0,0 -> (1,0), (1,1)
(1,0) -> (2,0), (2,1)
(1,1) -> (2,1), (2,2)
{% endhighlight %}
复杂度O(n^2)，n为所有的点的个数，求出了每个点到所有点的距离

3) 动态规划，思路：
{% highlight C++ %}
1
2 3
4 5 6
7 8 9 10

state: f[i][j]代表就是从i,j出发，到最底层的最短路
function: f[i][j] = min(f[i + 1][j], f[i + 1][j + 1]) + a[i][j]
intialize: f[n][x] = 0; //越界的那一层
answer: f[0][0]
{% endhighlight %}

**记忆化搜索**--动态规划最本质的思想:

* Advantage: Easy to think and implement
* Disadvantage: Expensive memory cost.

二维数组代码：
{% highlight C++ %}
int minimumTotal(vector<vector<int> > &triangle) {
    int size = triangle.size();
    if(size == 0)   return 0;
    //从下往上
    vector<vector<int> > dp(triangle);
    for(int i=size-2; i>=0; i--) {
        for(int j=0; j<=i; j++)
            dp[i][j] = min(dp[i+1][j], dp[i+1][j+1]) + triangle[i][j];
    }
    return dp[0][0];
}
{% endhighlight %}

压缩空间：

* 当第i层状态，只跟i-1有关，就可压缩成一维的。把i-2以前的都扔掉
* DP大类1：Two Sequence Dp 一般都可以压缩空间
* DP大类2：sequence一般不可以，貌似本来就是一维的(后边具体分析).
{% highlight C++ %}
int minimumTotal(vector<vector<int> > &triangle) {
    int size = triangle.size();
    if(size == 0)   return 0;
    //从下往上
    vector<int> dp(triangle[size-1]);   //用最后一行初始化!
    for(int i=size-2; i>=0; i--) {
        for(int j=0; j<=i; j++)
            dp[j] = min(dp[j], dp[j+1]) + triangle[i][j];  //从前往后，不会覆盖需要的
    }
    return dp[0];
}
{% endhighlight %}

### 2. Clues: 如何想到使用DP
* Can not sort.
* Find a maximum/minimum result
* Decide whether something is possible or not
* Count all possbile solutions: doesn’t care about the solution details, only the
count or possibility（若求所有的排列，只能全搜）

### 3. 动态规划的**4点要素**
* 状态 State (灵感，创造力，存储小规模问题的结果）
* 方程 Function (状态之间的联系，怎么通过小的状态，来算大的状态)
* 初始化 Intialization (最极限的小状态是什么)
* 答案 Answer (最大的那个状态是什么)
