---
layout: post
title:  "Basics on Dynamic Programming(DP) 5 - Palindrome"
date:   2016-03-20 14:30:00
tags: [algorithm, leetcode, dynamic programming, dp, palindrome]
categories: Algorithm
---

> 字符串处理，Palindrome

### 1. [Palindrome Partitioning II - Leetcode 132 - 边界很难](https://leetcode.com/problems/palindrome-partitioning-ii/)
```
Given a string s, partition s such that every substring of the partition is a palindrome.
Return the minimum cuts needed for a palindrome partitioning of s.

For example, given s = "aab",
Return 1 since the palindrome partitioning ["aa","b"] could be produced using 1 cut.
```
* state: f[i] "前i"个字符，用最少几次cut能分割为若干回文子串
* function: f[i] = min{f[j] + 1, j < i && **j+1 ~ i 是一个回文子串**}(找“最后”一次cut的位置) ，粗体部分也是DP可以用二维数组记录一下
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

### 2. [Longest Palindromic Substring - Leetcode 5](https://leetcode.com/problems/longest-palindromic-substring/)
```
Given a string S, find the longest palindromic substring in S.
You may assume that the maximum length of S is 1000, and there exists one unique longest palindromic substring.
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
另一种O(nlogn)的方法，使用后缀数组，将abcdcb镜面反转加上一个$符号，变为：abcdcb$bcdcba，于是将原问题转换为了求该字符串的所有后缀的最长公共前缀问题。实现较为复杂，可以[参考](https://www.douban.com/note/210945706/?type=like)和[这里](http://www.acmerblog.com/suffix-tree-6152.html)

### 3. 给定字符串，最少插入多少字符，变成回文
* state: f[i][j] 子串i~j变成回文，最少插入的字符数，
* function: f[i][j] = if s[i] = s[j]: f[i+1][j-1] else: (1+min{f[i][j-1], f[i+1][j]})
* intialize: f[i][i] = 0
* answer: f[0][n-1]
{% highlight C++ %}
for (i = n - 1; i >= 0; --i) {      // 倒着来，否则i+1没算出来
    for (j = i; j < n; ++j) {
        if (s[i] == s[j])
        	ans[i][j] = ans[i + 1][j - 1];  // 如果两个游标所指字符相同，向中间缩小范围
        else 
        	ans[i][j] = 1 + MIN(ans[i][j - 1], ans[i + 1][j]);  // 如果不同，典型的状态转换方程
    }
}
printf("%d\n", ans[0][n - 1]);
{% endhighlight %}
