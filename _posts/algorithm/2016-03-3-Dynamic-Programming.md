---
layout: post
title:  "Getting started with Dynamic Programming(DP)"
date:   2016-03-03 11:30:00
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
1) 原始的解法, 爆炸性复杂度2^n, 伪代码:
{% highlight C++ %}
void dfs(int x, int y, int sum) {
    if (x == n) {
        if (sum > best) {
            best = sum;
        }
        return;
    }
    
    if (max[x][y] != -1) {    //尝试最优性剪枝：自顶向下的优化方式
        if (max[x][y] <= sum) {
            return;
        }
    }
    max[x][y] = sum;
    
    dfs(x + 1, y, sum + a[x][y]);
    dfs(x + 1, y + 1, sum + a[x][y]);
}
 
best = -MAXINT;
dfs(0, 0, 0);
{% endhighlight %}
2) 此类方法的优化：
    1. 如果没解了，就不搜了（可行性剪枝） 不可行
    2. 如果解不可能最优，就不搜了（最优性剪枝）  可尝试
    3. 看看有没有重复计算, 伪代码如下：
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
当第i层状态，只跟i-1有关，就可压缩成一维的。把i-2以前的都扔掉
Two Sequence Dp一般都可以压缩空间；sequence一般不可以，貌似本来就是一维的(后边具体分析).
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
* Count all possbile solutions: problem doesn’t care about the solution details, only care about the
count or possibility（若求所有的排列，只能全搜，若求排列个数，可以有方程fn fn-1关系）

### 3. 动态规划的**4点要素**
* 状态 State (灵感，创造力，存储小规模问题的结果）
* 方程 Function (状态之间的联系，怎么通过小的状态，来算大的状态)
* 初始化 Intialization (最极限的小状态是什么)
* 答案 Answer (最大的那个状态是什么)

### 4. DP问题大类1: Matrix DP

#### 4.1 [Unique Paths - Leetcode 62](https://leetcode.com/problems/unique-paths/)

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

#### 4.2 [Unique Paths II - Leetcode 63](https://leetcode.com/problems/unique-paths-ii/)
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

#### 4.3 [Minimum Path Sum - Leetcode 64](https://leetcode.com/problems/minimum-path-sum/)
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

### 5. 大类2: Sequence Dp，一个序列，只考虑前i个

#### 5.1 [Climbing Stairs - Leetcode 70](https://leetcode.com/problems/climbing-stairs/)

```
You are climbing a stair case. It takes n steps to reach to the top.

Each time you can either climb 1 or 2 steps. In how many distinct ways can you climb to the top?
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
省空间的方法：因为只前2个数有用，所以不需要n的数组
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

#### 5.2 [Decode Ways - Leetcode 91](https://leetcode.com/problems/decode-ways/)

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

类似前 一道题，记录cur前2个的种类数，一直遍历到最后：
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

#### 5.3 [Jump Game - Leetcode 55](https://leetcode.com/problems/jump-game/)

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
    //另，之前的贪心方法，也可用动态规划 f[i] = max(f[i-1],A[i-1])-1,i>0 (表示，走到A[i]时，剩余的还能走的最大步数) 
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

#### 5.4 [Jump Game II - Leetcode 45](https://leetcode.com/problems/jump-game-ii/)

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

#### 5.5 [Palindrome Partitioning II - Leetcode 132- 边界很难](https://leetcode.com/problems/palindrome-partitioning-ii/)
```
Given a string s, partition s such that every substring of the partition is a palindrome.

Return the minimum cuts needed for a palindrome partitioning of s.

For example, given s = "aab",
Return 1 since the palindrome partitioning ["aa","b"] could be produced using 1 cut.
```
* state: f[i] "前i"个字符，用最少几次cut能分割为若干回文子串
* function: f[i] = min{f[j] + 1, j < i && j+1 ~ i 是一个回文子串}(找“最后”一次cut的位置) ，粗体部分也是DP可以用二维数组记录一下
* intialize: f[i] = i-1
* answer: f[s.length()]，因为第0个留出来了

{% highlight C++ %}
//所以需DP的方法 时间复杂度 O(n^2)，空间复杂度 O(n^2)
int minCut(string s) {
    int n = s.size();
    vector<vector<bool> > isPalindrome(n, vector<bool>(n, false));  //都是false
    vector<int> dp(n+1, 0); //"前i"个字符，用最少几次cut能分割为若干回文子串
    //initialize
    for(int i = 0; i <= n; i++)
        dp[i] = i-1;  //第一个f[0]=-1,f1=0
    
    for(int i=1; i<=n; i++) {   //从第2个字符开始，边界情况很麻烦
        for(int j=0; j<i; j++) {   //1个字符也算
            if(s[j] == s[i-1] && (i-j<3 || isPalindrome[j+1][i-2])) {
                isPalindrome[j][i-1] = true;
                dp[i] = min(dp[j]+1, dp[i]);  //是j-1之前的，加上1
            }
        }
    }
    return dp[n];
}
{% endhighlight %}

#### 5.5 [Longest Palindromic Substring - Leetcode 5](https://leetcode.com/problems/longest-palindromic-substring/)
```
Given a string S, find the longest palindromic substring in S. You may assume that the maximum length of S is 1000, and there exists one unique longest palindromic substring.
```
传统方法：
{% highlight C++ %}
string longestPalindrome(string s) {
    //从中间往2边扩散的方法(比从2边向中间走强!走的地方都是有用的)
    //并且回文串有2N-1(N N-1)个这样的中心（中心或者在任意一个字符或者在任意两个字符的之间）n^2
    string ret;
    int length = s.size();
    if(length <= 1) return s;
    for(int i=0; i<length-1; i++) { //1个或者2个字符，开始2边扩散! 扫一遍，所有的情况就都包括了
        string res = longestPalindromeCore(s, i, length);
        if(res.size() > ret.size())
            ret = res;
    }
    return ret;
}
string longestPalindromeCore(const string &s, int mid, int length) {
    int i;
    //单点扩散
    for(i=1; mid-i>=0 && mid+i<length; i++) {
        if(s[mid-i] != s[mid+i])
            break;
    }
    string midstr1(s, mid-i+1, 2*(i-1)+1);   //初始化为mid,从mid往2边搜索.注意这里的下标。最后一个参数是长度!!
    if(s[mid] != s[mid+1])
        return midstr1;
    //若该点和下个相同，则双点扩散
    for(i=1; mid-i>=0 && mid+1+i<length; i++) {
        if(s[mid-i] != s[mid+1+i])
            break;
    }
    string midstr2(s, mid-i+1, 2*i);
    return (midstr1.size()>midstr2.size()) ? midstr1 : midstr2;
}
{% endhighlight %}
DP方法：
{% highlight C++ %}
string longestPalindrome(string s) {
    //方法2: DP F(i,j) = F(i+1,j-1) if s[i] == s[j].其中F(i,j)被定义为子串s[i...j]是否是回文
    //所以枚举长度从1~N的子串(O(n2))，再判断是否为回文(O(1)).总体时间复杂度O(n2), 空间复杂度O(n2)。
    string ret;
    int length = s.size();
    if(length <= 1) return s;
    
    bool isPalindrome[1000][1000] = {false};
    int max_len = 1, start = 0; // 最长回文子串的长度，起点
    for(int i=0; i<length; i++) {
        isPalindrome[i][i] = true;
        for(int j=0; j<i; j++) { //[j][i]是否是回文  相当于下三角  或者j=i，j<length
            if(s[j] == s[i] && (i-j < 2 || isPalindrome[j+1][i-1])) {   //只有这样，j，i才是回文
                isPalindrome[j][i] = true; //从小到大
                if(max_len < i-j+1){
                    max_len = i-j+1;
                    start = j;
                }
            }
        }
    }
    return s.substr(start, max_len);
}
{% endhighlight %}

d.3, 给定一个字符串，最少插入多少字符，变成回文串
for (i = n - 1; i >= 0; --i) {      //倒着来，否则i+1没算出来
    for (j = i; j < n; ++j) {
        if (s[i] == s[j]) ans[i][j] = ans[i + 1][j - 1];//如果两个游标所指字符相同，向中间缩小范围
        else ans[i][j] = 1 + MIN(ans[i][j - 1], ans[i + 1][j]);//如果不同，典型的状态转换方程
    }
}
printf("%d\n", ans[0][n - 1]);

e, Word Break - Leetcode - 同样注意边界

Given a string s and a dictionary of words dict, determine if s can be segmented into a space-separated sequence of one or more dictionary words.

For example, given
s = "leetcode",
dict = ["leet", "code"].

Return true because "leetcode" can be segmented as "leet code".

代码：
bool wordBreak(string s, unordered_set<string> &dict) {
    //DP[i]代表，s[0,i) 是否可以分词
    //方程dp[i] = dp[j] && j+1~i属于dict
    int n = s.size();
    vector<bool> dp(n+1, false); //初始化，长度为 n 的字符串有 n+1 个隔板
    dp[0] = true;   //这样是为了减少0位置的判断,空字符串
    for(int i=1; i<=n; i++) {
        for(int j=0; j<i; j++) {
            if(dp[j] && dict.find(s.substr(j, i-j))!=dict.end()) {  //从j开始算的
                dp[i] = true;
                break;
            }
        }
    }
    return dp[n];
}

e.2, Word Break II - Leetcode

Given a string s and a dictionary of words dict, add spaces in s to construct a sentence where each word is a valid dictionary word.

Return all such possible sentences. ==>这种题目，不能用DP，只能DFS

For example, given
s = "catsanddog",
dict = ["cat", "cats", "and", "sand", "dog"].

A solution is ["cats and dog", "cat sand dog"].
DFS原始代码会超时：
vector<string> wordBreak(string s, unordered_set<string> &dict) {
    vector<string> res;
    DFS(s, 0, dict, "", res);
    return res;
}
void DFS(string s, int index, unordered_set<string> &dict, string path, vector<string> &res) {
    if(index == s.size()) {
        res.push_back(path);
        return;     //string path恢复原状
    }
    for(int i=index; i<s.size(); i++) {
        if(dict.find(s.substr(index, i-index+1)) != dict.end()) {
            path += (path=="") ? "" : " ";  //是否有空格
            path += s.substr(index, i-index+1);  //有漏洞，需要完事儿 减去这个，否则重复了san还有sand都算进去了
            DFS(s, i+1, dict, path, res);
        }
    }
}
优化：重复计算太多，可以借鉴DP，存储所有的合法的单词 - 难!!
vector<string> wordBreak(string s, unordered_set<string> &dict) {
    //先用1的方法得到结果的组数，再分别求结果
    vector<vector<bool> > prev(s.length(), vector<bool>(s.length()+1, false));//prev[i][j]表示s[i, j)是一个合法单词,可以从j处切开
    vector<bool> dp(s.size() + 1, false);   //初始化为false!
    dp[0] = true; // 空字符串
    for(int i=1; i<=s.size(); i++)
        for(int j=i-1; j>=0; j--) {
            if(dp[j] && dict.find(s.substr(j, i-j))!=dict.end()) {
                dp[i] = true;
                prev[j][i] = true;
            }
        }
    vector<string> result;
    if(!dp[s.size()])   return result;
    gen_path(s, prev, 0, "", result);
    reverse(result.begin(), result.end());
    return result;
}
// DFS 遍历树，生成路径
void gen_path(string s, const vector<vector<bool> > &prev, int cur, string path, vector<string> &result) {
    if(cur == s.size()) {   //找到，存入
        result.push_back(path);
        return;  //string path恢复原状
    }   
    for(int i=cur+1; i<=s.size(); i++) {  //找从cur开始的，可行的解!
        if(prev[cur][i]) {
            path += (path=="") ? "" : " ";  //是否有空格
            path += s.substr(cur, i-cur);
            gen_path(s, prev, i, path, result); //这里还是i!
            path = path.substr(0, path.size()-i+cur);
            if(path != "")  path = path.substr(0, path.size()-1);
        }
    }
}

f，Longest Increasing Subsequence  LIS 子序列

http://www.lintcode.com/en/problem/longest-increasing-subsequence/

Given a sequence of integers, find the longest increasing subsequence (LIS).

You code should return the length of the LIS.
Example

For [5, 4, 1, 2, 3], the LIS  is [1, 2, 3], return 3

For [4, 2, 4, 5, 3, 7], the LIS is [4, 4, 5, 7], return 4
 
state: f[i]表示[前i]个数字中，以[第i]个数为结尾的LIS最长是多少
function: f[i] = max{f[i], f[j] + 1, j < i && nums[j] <= nums[i]}
intialize: f[0..n-1] = 1   ，变化
answer: max(f[0..n-1])，变化，所有的点都是终点。

代码：
int longestIncreasingSubsequence(vector<int> nums) {
    //write your code here
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
另外的，O(nlogn)的算法：
http://blog.csdn.net/yorkcai/article/details/8651895
贪心 + 二分搜索：
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
求出序列：用另外的数组记录前一个节点的位置
http://prismoskills.appspot.com/lessons/Dynamic_Programming/Chapter_06_-_Max_increasing_subsequence.jsp

注意：当出现1，5，8，2这种情况时，栈内最后的数是1，2，8不是正确的序列，难道错了？

分析一下，我们可以看出，虽然有些时候这样得不到正确的序列，但最后算出来的个数是没错的，为什么呢？

想想，当a[i]>top时，总个数直接加1，这肯定没错；但当a[i]<top时呢？ 这时a[i]会替换栈里面的某一个元素，大小不变，就是说一个小于栈顶的元素加入时，总个数不变。

这两种情况的分析可以看出，如果只求个数的话，这个算法比较高效；但如果要求打印出序列时，就只能用动态规划了。

6，Two Sequence Dp，一个序列，只考虑前i个
a, Longest Common Subsequence  LCS - lintcode
http://www.lintcode.com/en/problem/longest-common-subsequence/

Given two strings, find the longest comment subsequence (LCS).

Your code should return the length of LCS.
Example

For "ABCD" and "EDCA", the LCS is "A" (or D or C), return 1

For "ABCD" and "EACB", the LCS is "AC", return 2


state: f[i][j] 第一个字符串的前i个字符匹配上第二个字符串的前j个字符，所能找到的LCS的长度是什么

function: f[i][j] = f[i-1][j-1] + 1 // a[i] == b[j]

                  = max(f[i-1][j], f[i][j-1]) // a[i] != b[j]

intialize: f[0][i] = 0, f[i][0] = 0        类似二维matrix的，初始化两边

answer: f[a.length()][b.length()]

代码：
int longestCommonSubsequence(int *a, int m, int*b, int n) {
    vector<vector<int> > dp(m+1, vector<int>(n+1, 0));    //initialize with 0，因为边界，多一个搞
    for(int i=1; i<=m; i++) {
        for(int j=1; j<=n; j++) {
            if(a[i-1] == b[j-1])
                dp[i][j] = dp[i-1][j-1] + 1;
            else
                dp[i][j] = max(dp[i][j-1], dp[i-1][j]);
        }
    }
    return dp[m][n]; 
}
压缩后，感觉有点问题，不是只跟i-1有关, 需要变量保存i-1,j-1
int longestCommonSubsequence2(int *a, int m, int*b, int n) {
    vector<int> dp(n+1, 0); //initialize with 0，因为边界，多一个搞
    int lastDp=0;
    for(int i=1; i<=m; i++) {
        for(int j=1; j<=n; j++) {
            int temp = dp[j];
            if(a[i-1] == b[j-1])
                dp[j] = lastDp + 1;
            else
                dp[j] = max(dp[j-1], dp[j]);  //dp[j]相当于dp[i-1][j], 而dp[j-1]相当于dp[i][j-1]
            lastDp = temp;
        }
    }
    return dp[n]; 
}


若是连续子序列：阿里笔试题目
int longestCommonSubstring(string query, string text)
{
    int m = query.size();
    int n = text.size();
    int maxLen = 0;
    vector<vector<int> > dp(m+1, vector<int>(n+1, 0));  //初始化，都是0
    for(int i=1; i<=m; i++) {
        for(int j=1; j<=n; j++) {
            dp[i][j] = (query[i-1] != text[j-1]) ? 0 : (dp[i-1][j-1] + 1);
            maxLen = maxLen < dp[i][j] ? dp[i][j] : maxLen;
        }
    }
    return maxLen;
}
压缩后：
int longestCommonSubstring(string query, string text)
{
    int m = query.size();
    int n = text.size();
    int maxLen = 0;
    vector<int> dp(n+1, 0);  //初始化，都是0
    for(int i=1; i<=m; i++) {
        for(int j=n; j>0; j--) {
            dp[j] = (query[i-1] != text[j-1]) ? 0 : (dp[j-1] + 1);
            maxLen = maxLen < dp[j] ? dp[j] : maxLen;
        }
    }
    return maxLen;
}


b，Edit Distance - 和上题很类似！ - Leetcode

Given two words word1 and word2, find the minimum number of steps required to convert word1 to word2. (each operation is counted as 1 step.)

You have the following 3 operations permitted on a word:

a) Insert a character
b) Delete a character
c) Replace a character


state: f[i][j] 第一个字符串的前i个字符匹配上第二个字符串的前j个字符，最少编辑距离

function: f[i][j] = f[i-1][j-1] // a[i] == b[j]

                  = min(f[i-1][j], f[i][j-1], f[i-1][j-1]) + 1 // a[i] != b[j]

intialize: f[0][i] = i, f[i][0] = i

answer: f[a.length()][b.length()]


int minDistance(string word1, string word2) {
    int m=word1.size(), n=word2.size();
    vector<vector<int> > dp(m+1, vector<int>(n+1, INT_MAX));    
    for(int i=0; i<=m; i++)     //初始化,有0个
        dp[i][0] = i;
    for(int i=0; i<=n; i++)     //初始化
        dp[0][i] = i;
         
    for(int i=1; i<=m; i++) {
        for(int j=1; j<=n; j++) {
            if(word1[i-1] == word2[j-1])
                dp[i][j] = dp[i-1][j-1];
            else
                dp[i][j] = min(min(dp[i][j-1], dp[i-1][j]), dp[i-1][j-1]) + 1;
        }
    }
    return dp[m][n];
}
压缩一维: 注意保存i-1 j-1
int minDistance(string word1, string word2) {
    int m=word1.size(), n=word2.size();
    vector<int> dp(n+1, INT_MAX);    
    for(int i=0; i<=n; i++)     //初始化
        dp[i] = i;
     
    int lastDp=0;           //额外用一个变量记录 f[i-1][j-1]    
    for(int i=1; i<=m; i++) {
        lastDp = dp[0]; //每次初始化为第一个
        dp[0] = i;
        for(int j=1; j<=n; j++) {
            int temp = dp[j];
            if(word1[i-1] == word2[j-1])
                dp[j] = lastDp;
            else
                dp[j] = min(min(dp[j-1], dp[j]), lastDp) + 1;
            lastDp = temp;
        }
    }
    return dp[n];
}


c，Interleaving String - Leetcode

Given s1, s2, s3, find whether s3 is formed by the interleaving of s1 and s2.

For example,
Given:
s1 = "aabcc",
s2 = "dbbca",

When s3 = "aadbbcbcac", return true.
When s3 = "aadbbbaccc", return false.

state: f[i][j]第一个字符串的前i个字符，匹配上第二个字符串的前j个字符，能否交错构成第三个字符串的前i+j个字符

function: f[i][j] = f[i-1][j] // if a[i] == c[i+j]   再看一下？

                 || f[i][j-1] // if b[j] == c[i+j]

intialize: f[i][0]=a的前i个字符=c的前i个字符。f[0][i]=同理可得

answer: f[a.length()][b.length()]


代码：
bool isInterleave(string s1, string s2, string s3) {
    if (s3.length() != s1.length() + s2.length())   //直接错误
        return false;
    int m=s1.length(), n=s2.length();
    vector<vector<bool>> dp(m+1, vector<bool>(n+1, false));
    //初始化，2条边
    dp[0][0] = true;
    for(int i = 1; i <= m; i++) {
        if(s1[i-1] == s3[i-1])  dp[i][0] = true;
        else    break;  //只要有1个不等于，就break
    }
    for(int i = 1; i <= n; i++) {
        if(s2[i-1] == s3[i-1])  dp[0][i] = true;
        else    break;  //只要有1个不等于，就break
    }
    //开始算
    for (int i = 1; i <= m; i++) {
        for(int j = 1; j <= n; j++) {
            dp[i][j] = ((s1[i-1] == s3[i+j-1]) && dp[i-1][j])
                    || ((s2[j-1] == s3[i+j-1]) && dp[i][j-1]);
        }
    }
    return dp[m][n];
}
压缩：
稍微复杂，以后再说，pdf中有答案
string longestPalindrome(string s) {
    //方法2: DP F(i,j) = F(i+1,j-1) if s[i] == s[j].其中F(i,j)被定义为子串s[i...j]是否是回文
    //所以枚举长度从1~N的子串(O(n2))，再判断是否为回文(O(1)).总体时间复杂度O(n2), 空间复杂度O(n2)。
    string ret;
    int length = s.size();
    if(length <= 1) return s;
     
    bool isPalindrome[1000][1000] = {false};
    int max_len = 1, start = 0; // 最长回文子串的长度，起点
    for(int i=0; i<length; i++) {
        isPalindrome[i][i] = true;
        for(int j=0; j<i; j++) {
            if(s[j] == s[i] && (i-j < 2 || isPalindrome[j+1][i-1])) {   //只有这样，j，i才是回文
                isPalindrome[j][i] = true;  //从小到大
                if(max_len < i-j+1){
                    max_len = i-j+1;
                    start = j;
                }
            }
        }
    }
    return s.substr(start, max_len);
}

e，Distinct Subsequence

Given a string S and a string T, count the number of distinct subsequences of T in S.

A subsequence of a string is a new string which is formed from the original string by deleting some (can be none) of the characters without disturbing the relative positions of the remaining characters. (ie, "ACE" is a subsequence of "ABCDE" while "AEC" is not).

Here is an example:
S = "rabbbit", T = "rabbit"

Return 3.

state: f[i][j]第一个字符串的前i个字符，匹配上第二个字符串的前j个字符，有多少个distinct subseqs

function: f[i][j] = f[i-1][j] + f[i-1][j-1] // a[i] == b[j]

                  = f[i-1][j] // a[i] != b[j]  没有删除j的这种情况了

intialize: f[i][0]=1, f[0][i] = 0 (i>0)

answer: f[a.length()][b.length()]

代码：
int numDistinct(string S, string T) {
    //dfs做，大数据会超时；题目意思，S中怎么删除，能变到T；用DP
    int M=S.size(), N=T.size();
    if(M == 0 && N == 0)    return 1;
    else if(M < N)  return 0;
     
    vector<vector<int> > dp(M+1, vector<int>(N+1, 0));
    for(int i=0; i<=M; i++)   //初始化
        dp[i][0] = 1;
    for(int i=1; i<=N; i++) //0,0是1!!
        dp[0][i] = 0;
    for (int i = 1; i <= M; i++) {
        for (int j = 1; j <= N; j++) {
            if (S[i-1] == T[j-1])   //若 S[i]==T[j]，则可以使用 S[i]，是二者的和
                dp[i][j] = dp[i-1][j] + dp[i-1][j-1];
            else
                dp[i][j] = dp[i-1][j];  //不能使用S[i]，则等于i-1到j的种类数
        }
    }
    return dp[M][N];
}
压缩一维： 不太好理解
int numDistinct(string S, string T) {
    int M=S.size(), N=T.size();
    if(M == 0 && N == 0)    return 1;
    else if(M < N)  return 0;
    vector<int> dp(N+1, 0);
    dp[0] = 1;
    for (int i = 0; i < M; i++) {
        for (int j = N-1; j >= 0; j--) {    //压缩后，从后往前，避免覆盖
            if (S[i] == T[j])   //若 S[i]==T[j]，则可以使用 S[i]，是二者的和
                dp[j+1] = dp[j] + dp[j+1];
        }
    }
    return dp[N];
}


7，Interval DP 区间内动态规划 -- 不掌握也可以，考的少

a，Merge Stone

http://wikioi.com/problem/1048/

有n堆石子排成一列，每堆石子有一个重量w[i], 每次合并可以合并相邻的两堆石子，一次合并的代价为两堆石子的重量和w[i]+w[i+1]。问安排怎样的合并顺序，能够使得总合并代价达到最小。

4 1 1 4

--- ---

 5 5 --- 10

------

 10  --- 10

          = 20

         

4 1 1 4

  ---

   2       2 + 6+ 10 = 18

   ----

     6

------

   10


state: f[i][j]代表了从i合并到j的最小代价是什么

function: f[i][j] = min(f[i][k] + f[k+1][j] + sum[i][j])，sum是一定的，从k中找最小的

intialize: f[i][i] = 0

answer: f[1][n]

for(int len=1; len<n; len++)
{
    for(int i=1; i+len<=n; i++)
    {
        int end = i+len;
        int tmp = INF;
        int k = 0;
        for(int j=p[i][end-1]; j<=p[i+1][end]; j++)
        {
            if(dp[i][j] + dp[j+1][end] + sum[end] - sum[i-1] < tmp)
            {
                tmp = dp[i][j] + dp[j+1][end] + sum[end] - sum[i-1];
                k = j;
            }
        }
        dp[i][end] = tmp;
        p[i][end] = k;
    }
}

8，群里讨论：

给出下面这个图 设计数据结构和算法求出图中所有的正方形. 整个过程没有给任何hint. 这题应该怎么考虑?
技巧就是从每一个点开始存储上下左右四个方向最多延伸到的位置
 这个题我感觉是得一个个数
枚举右下角位置
然后枚举正方形边长
根据预处理的延伸情况判断是否能够有一个正方形被构造出来
复杂度O(n^3)
 预处理可以O(n^2)
预处理是有递推关系的
【传说】黄老师(276810220)  5:48:52
但是后面枚举的部分，只能O(n^3)
这个题目不能动态规划的原因是：他给定了一个可以变化的图，这个图上规模为n的图和规模为n-1的图中正方形个数之间不存在递推关系 

一般来说处理矩阵的问题，大部分都是O(n^3)
 大家可以想一下某个经典面试题：求n*n矩阵的最大矩阵。矩阵中的每个数可以正可以负。（Leetcode最大和Maximum Subarray的升级版 ）
这个的极限复杂度也是O(n^3)
http://blog.csdn.net/beiyeqingteng/article/details/7056687
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
有一个可以O(n^2)的题目是这样的：给一个01矩阵，问最大的全1正方形。
Maximal Rectangle - Leetcode - 难
Given a 2D binary matrix filled with 0's and 1's, find the largest rectangle containing all ones and return its area.
解法：http://blog.csdn.net/lanxu_yy/article/details/17533203
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
 
还有一种问法是：问最大的对角线是1，其他部分是0的矩阵。
大家可以想想看我刚才提的这三个矩阵相关的问题怎么做。
提示都是用动态规划
 传说】黄老师(276810220)  11:43:38
longest palindrome 的算法比较常见的是O(nlogn)的方法，使用后缀数组，将abcdcb镜面反转加上一个$符号，变为：abcdcb$bcdcba，于是将原问题转换为了求该字符串的所有后缀的最长公共前缀问题。
【传说】黄老师(276810220)  11:44:42
如果面试问到这个问题，可以从三个层次来答：首先答出朴素的算法。如果他不够满意然后答出O(nlogn)的后缀数组算法，如果还不够满意，就告诉她上面那篇论文的方法。
一般来说你说个暴力的估计就过了……

9, Yahoo面试题：
n个人排名，允许并列名次，共有多少种排名结果
假设n个人，排出了m个名次，有f(n,m)种结果（1 <=m <=n） 
当m=1 
f(n,m)=1 
当n <m 
f(n,m)=0 
当1<m <=n 
假设n-1个人，排出了m个名次；新来1人，与前面某名次并列，有f(n-1,m)*m种结果 
假设n-1个人，排出了m个名次；新来1人，与前面名次都不并列，有f(n-1,m-1)*m种结果 
f(n,m)= f(n-1,m)*m + f(n-1,m-1)*m 

综合上述，递推式: 
f(n,m) = 0 , n <m||m <1 
f(n,m) = 1 , 1=m <=n 
f(n,m) = (f(n-1,m) + f(n-1,m-1))*m 1 <m <=n 

n个人的排名就是f(n,1)+f(n,2)+f(n,3)+...+f(n,n) 

10. 0-1背包问题
    vector<int> dp(M+1, 0); //手里奖券数，能买的最大的价值
    for(int i=0; i<N; i++) { //在执行i次循环后(此时已经处理i个物品)，前i个物体放到容量v时的最大价值，即之前的f[i][v]
        for(int j=M; j>=need[i]; j--) //现在有的钱
            dp[j] = max(dp[j], dp[j-need[i]]+val[i]);    //取不选，或选的最大值
    }
    cout << dp[M];
背包相关：
http://www.lintcode.com/zh-cn/problem/backpack/
http://www.lintcode.com/zh-cn/problem/minimum-adjustment-cost/
http://www.lintcode.com/zh-cn/problem/k-sum/

11，给一个 target number, 比如说9, 可以被表示成 sum of square.
http://www.cnblogs.com/skyivben/archive/2011/11/26/2264679.html
比如 1+1+1 ... +1 = 1+4+4= 9 找出最短的list 
背包DP: F[i][T]表示用1^2,2^2,3^2..i^2 组成和为T的最短长度 
F[i][T] = Min(F[i-1][T], F[i-1][T-k*i^2] + 1,  1<=k<=不让第二维越界)
DP可以记录方案, DP能记录某一种最佳方案，记录不了所有最佳方案
