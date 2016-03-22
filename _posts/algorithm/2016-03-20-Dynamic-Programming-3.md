---
layout: post
title:  "Basics on Dynamic Programming(DP) 3 - Sequence DP"
date:   2016-03-20 12:30:00
tags: [algorithm, leetcode, dynamic programming, dp]
categories: Algorithm
---

> 第2类常见DP问题：Sequence DP，一个序列，第i的状态只考虑前i个

### 1. [Climbing Stairs - Leetcode 70](https://leetcode.com/problems/climbing-stairs/)

```
You are climbing a stair case. It takes n steps to reach to the top.
Each time you can either climb 1 or 2 steps.
In how many distinct ways can you climb to the top?
```

* state: f[i]代表"前"i层楼梯，走到了第i层楼梯的方案数
* function: f[i] = f[i - 1] + f[i - 2]
* intialize: f[0] = 1
* answer: f[n]

{% highlight C++ %}
int climbStairs(int n) {
     vector<int> dp(n+1);       //注意是从0到n，一共n+1个
     dp[0] = 1;
     dp[1] = 1;
     for(int i=2; i<=n; i++)    //DP就是都记录下来，存入数组，避免重复子结构的重复运算
        dp[i] = dp[i-1] + dp[i-2];
     return dp[n];
}
{% endhighlight %}
压缩空间：因为只前2个数有用，所以不需要n的数组
{% highlight C++ %}
int climbStairs(int n) {
     if(n <= 1) return n;
     int pre1=1, pre2=1, cur=0;
     for(int i=2; i<=n; i++) {
        cur = pre1 + pre2;
        pre1 = pre2;
        pre2 = cur;
     }
     return cur;
} 
{% endhighlight %}

### 2. [Decode Ways - Leetcode 91](https://leetcode.com/problems/decode-ways/)

```
A message containing letters from A-Z is being encoded to numbers using the following mapping:

'A' -> 1
'B' -> 2
...
'Z' -> 26

Given an encoded message containing digits, determine the total number of ways to decode it.

For example,
Given encoded message "12", it could be decoded as "AB" (1 2) or "L" (12).

The number of ways decoding "12" is 2.
```

类似前一题，记录cur前2个的种类数，一直遍历到最后：
{% highlight C++ %}
int numDecodings(string s) {
    int n = s.size();
    if(s[0] == '0')   return 0;  //special case!!!
    if(n <= 1)  return n;   
    int pre1=1, pre2=1, cur=0;  //顺序：pre1->pre2->cur
    for(int i=1; i<n; i++) {    //从第2个字符开始判断
        if(s[i] == '0') {   //特殊，只依赖于前一个
            if(s[i-1] == '1' || s[i-1] == '2')  //注意这么写得了!!!
                cur = pre1;
            else
                return 0;   //30,00,是没有办法解析的，直接错误
        } else if(s[i-1] == '1' || (s[i-1] == '2' && s[i] <= '6') ) {  //能连起来的情况
            cur = pre1 + pre2;
        } else {
            cur = pre2;
        }
        pre1 = pre2;
        pre2 = cur;
    }
    return cur;
}
{% endhighlight %}

### 3. [Jump Game - Leetcode 55](https://leetcode.com/problems/jump-game/)

```
Given an array of non-negative integers, you are initially positioned at the first index of the array.
Each element in the array represents your maximum jump length at that position.

Determine if you are able to reach the last index.

For example:
A = [2,3,1,1,4], return true.
A = [3,2,1,0,4], return false.
```

下面这个递推式子不太好，实现较复杂：

* state: f[i] 代表前i个格子，跳到了第i个格子，是否可行
* function: f[i] = OR(f[j], j < i && j可以跳到i)
* intialize: f[0] = true;
* answer: f[n-1]

贪心算法：好简洁啊
{% highlight C++ %}
bool canJump(int A[], int n) {
    //改进，只从前到后扫一遍，用一个max变量记录最远能到的地方
    //时间复杂度 O(n)，空间复杂度 O(1)
    int maxreach=1;
    for(int i=0; i<maxreach && maxreach<n; i++) //maxreach若>=n了，则不用继续找了
        maxreach = max(maxreach, i+1+A[i]);
    return maxreach >= n;   //是否到
}
{% endhighlight %}
DP：
{% highlight C++ %}
bool canJump(int A[], int n) {
    //另，之前的贪心方法，也可用动态规划
    //想到该方程难：f[i] = max(f[i-1],A[i-1])-1,i>0 (表示，走到A[i]时，剩余的还能走的最大步数) 
    //时间复杂度 O(n)，空间复杂度 O(n)
    vector<int> f(n, 0);
    f[0] = A[0]; //initialize
    for(int i=1; i<n; i++) {
        f[i] = max(f[i-1], A[i-1]) - 1;
        if(f[i] < 0)    return false;
    }
    return f[n-1] >= 0;   //是否到
}
{% endhighlight %}

### 4. [Jump Game II - Leetcode 45](https://leetcode.com/problems/jump-game-ii/)

```
Given an array of non-negative integers, you are initially positioned at the first index of the array.
Each element in the array represents your maximum jump length at that position.

Your goal is to reach the last index in the minimum number of jumps.

For example:
Given array A = [2,3,1,1,4]

The minimum number of jumps to reach the last index is 2. (Jump 1 step from index 0 to 1, then 3 steps to the last index.)
```
* state: f[i]从第0个位置跳到"第i"个位置时，最少的跳跃次数（跳过了前i个位置)
* function: f[i] = min{f[j] + 1, j < i && j 能够跳到 i)
* intialize: f[0] = 0;
* answer: f[n-1];

超时的方法，不好:
{% highlight C++ %}
int jump(int A[], int n) {
    vector<int> dp(n,0); //初始化dp[0]=0;
    for(int i=1; i<n; i++) {    //n^2复杂度
        int minStep = INT_MAX;
        for(int j=0; j<i; j++) {
            if(A[j]+j >= i)   
                minStep = min(minStep, dp[j]+1);
        } 
        dp[i] = minStep;
    }
    return dp[n-1];
}
{% endhighlight %}
另外方法：跳1步，能到的地方，类似滑动窗口:
{% highlight C++ %}
int jump(int A[], int n) {
    //贪心法，时间复杂度 O(n)，空间复杂度 O(1)
    //另一种方法，用一个[left, right],表示当前能覆盖的区间，即滑动窗口 ==>优化for循环逻辑
    int step = 0; // 最小步数
    int left = 0;
    int right = 0;
    if (n == 1) return 0;
    while (left <= right) { // 尝试从每一层跳最远，就是每走一步，就有个滑动窗口，算一遍能到的最大距离
        ++step;
        int old_right = right;
        for (int i = left; i <= old_right; ++i) {
            right = max(right, i + A[i]);
            if(right >= n-1)    return step;
        }
        left = old_right + 1;
    }
    return INT_MAX; //没找到
}
{% endhighlight %}

### 5. [Longest Increasing Subsequence - LIS 子序列](http://www.lintcode.com/en/problem/longest-increasing-subsequence/)

```
Given a sequence of integers, find the longest increasing subsequence (LIS).
You code should return the length of the LIS.

Example: 
For [5, 4, 1, 2, 3], the LIS  is [1, 2, 3], return 3
For [4, 2, 4, 5, 3, 7], the LIS is [4, 4, 5, 7], return 4
```
* state: f[i]表示[前i]个数字中，以[第i]个数为结尾的LIS最长是多少
* function: f[i] = max{f[i], f[j] + 1, j < i && nums[j] <= nums[i]}
* intialize: f[0..n-1] = 1, 变化
* answer: max(f[0..n-1]), 变化，所有的点都可能是终点。
{% highlight C++ %}
int longestIncreasingSubsequence(vector<int> nums) {
    int n = nums.size();
    vector<int> res(n, 1);  //initialize with 1,代表[前i]个数字中，以[第i]个数为结尾的LIS最长是多少
    int maxLen = 1;
    for(int i=1; i<n; i++) {
        for(int j=0; j<i; j++) {
            if(nums[j] <= nums[i])
                res[i] = max(res[i], res[j]+1);
        }
        maxLen = max(maxLen, res[i]);
    }
    return maxLen; 
}
void main()
{
    int A[5] = {5,4,1,2,3};
    cout << longestIncreasingSubsequence(vector<int>(A, A+5));
}
{% endhighlight %}
另外，[O(nlogn)的算法](http://blog.csdn.net/yorkcai/article/details/8651895), 贪心 + 二分搜索：
{% highlight C++ %}
int lis_greedy(int seq[], int n) {
    int top = 0;
    int* stack = new int[n];
    for (int i = 0; i < n; ++i) {
        stack[i] = 0;
    }
    stack[top] = seq[0];
    for (int i = 0; i < n; ++i) {
        if (seq[i] > stack[top]) {
            //如果seq[i]大于栈顶元素，则入栈
            stack[++top] = seq[i];
        } else {
            //从栈底开始，找到第一个>=seq[i]的元素所在位置
            int replace = binary_search(stack, seq[i], 0, top);
            stack[replace] = seq[i];
        }
    }
    delete[] stack;
    return top + 1;
}
{% endhighlight %}
注意：当出现1，5，8，2这种情况时，栈内最后的数是1，2，8不是正确的序列

分析一下，我们可以看出，虽然有些时候这样得不到正确的序列，但最后算出来的个数是没错的

当a[i]>top时，总个数直接加1，这肯定没错；但当a[i]<top时呢？这时a[i]会替换栈里面的某一个元素，大小不变，就是说一个小于栈顶的元素加入时，总个数不变。

这两种情况的分析可以看出，如果只求个数的话，这个算法比较高效；但如果要求打印出序列时，就只能用动态规划了。

若要求出序列，有另一种方法：[用另外的数组记录前一个节点的位置](http://prismoskills.appspot.com/lessons/Dynamic_Programming/Chapter_06_-_Max_increasing_subsequence.jsp)
