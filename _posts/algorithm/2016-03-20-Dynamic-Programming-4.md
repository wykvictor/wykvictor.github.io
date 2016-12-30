---
layout: post
title:  "Basics on Dynamic Programming(DP) 4 - Two Sequence DP"
date:   2016-03-20 13:30:00
tags: [algorithm, leetcode, dynamic programming, dp, two sequence]
categories: Algorithm
---

> 第3类DP问题：Two Sequence DP，2个序列相互关系

### 1. [Longest Common Subsequence  LCS - lintcode](http://www.lintcode.com/en/problem/longest-common-subsequence/)
```
Given two strings, find the longest comment subsequence (LCS).
Your code should return the length of LCS.

Example:
For "ABCD" and "EDCA", the LCS is "A" (or D or C), return 1
For "ABCD" and "EACB", the LCS is "AC", return 2
```

* state: f[i][j]  // 第一个字符串的前i个字符匹配上第二个字符串的前j个字符，所能找到的LCS的长度是什么
* function: f[i][j] = {f[i-1][j-1] + 1 (a[i] == b[j])} or {max(f[i-1][j], f[i][j-1]) (a[i] != b[j])}
* intialize: f[0][i] = 0, f[i][0] = 0  // 类似二维matrix的，初始化两边
* answer: f[a.length()][b.length()]

{% highlight C++ %}
int longestCommonSubsequence(string A, string B) {
  int m = A.size(), n = B.size();
  // initialize with 0，因为边界，多一个, 表示前0个字符
  vector<vector<int> > dp(m + 1, vector<int>(n + 1, 0));
  for (int i = 1; i <= m; i++) {
    for (int j = 1; j <= n; j++) {
      if (A[i - 1] == B[j - 1])
        dp[i][j] = dp[i - 1][j - 1] + 1;
      else
        dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
    }
  }
  return dp[m][n];
}
{% endhighlight %}
压缩后，注意！不是只跟i-1有关, 需要变量保存i-1,j-1
{% highlight C++ %}
int longestCommonSubsequence(string A, string B) {
  int m = A.size(), n = B.size();
  vector<int> dp(n + 1, 0);  // initialize with 0
  int lastDp = 0;
  for (int i = 1; i <= m; i++) {
    for (int j = 1; j <= n; j++) {
      int temp = dp[j];
      if (A[i - 1] == B[j - 1])
        dp[j] = lastDp + 1;
      else
        dp[j] = max(dp[j - 1], dp[j]);
      // dp[j]相当于dp[i-1][j], 而dp[j-1]相当于dp[i][j-1]
      lastDp = temp;
    }
  }
  return dp[n];
}
{% endhighlight %}
变体，连续，子序列，更简单些
{% highlight C++ %}
int longestCommonSubstring(string query, string text) {
  int m = query.size();
  int n = text.size();
  int maxLen = 0;
  vector<vector<int> > dp(m + 1, vector<int>(n + 1, 0));  //初始化，都是0
  for (int i = 1; i <= m; i++) {
    for (int j = 1; j <= n; j++) {
      // 这里不同，为0
      dp[i][j] = (query[i - 1] != text[j - 1]) ? 0 : (dp[i - 1][j - 1] + 1);
      maxLen = maxLen < dp[i][j] ? dp[i][j] : maxLen;  // 这里不同，一直保存max
    }
  }
  return maxLen;
}
{% endhighlight %}
压缩后：
{% highlight C++ %}
int longestCommonSubstring(string query, string text) {
  int m = query.size();
  int n = text.size();
  int maxLen = 0;
  vector<int> dp(n + 1, 0);  //初始化，都是0
  for (int i = 1; i <= m; i++) {
    for (int j = 1; j <= n; j++) {
      dp[j] = (query[i - 1] != text[j - 1]) ? 0 : (dp[j - 1] + 1);
      maxLen = maxLen < dp[j] ? dp[j] : maxLen;
    }
  }
  return maxLen;
}
{% endhighlight %}

### 2. [Edit Distance - Leetcode 72](https://leetcode.com/problems/edit-distance/)
```
Given two words word1 and word2, find the minimum number of steps required to convert word1 to word2. (each operation is counted as 1 step.)

You have the following 3 operations permitted on a word:
a) Insert a character
b) Delete a character
c) Replace a character
```

* state: f[i][j] 第一个字符串的前i个字符匹配上第二个字符串的前j个字符，最少编辑距离
* function: f[i][j] = {f[i-1][j-1] (a[i] == b[j])} or {min(f[i-1][j], f[i][j-1], f[i-1][j-1]) + 1 (a[i] != b[j])}
* intialize: f[0][i] = i, f[i][0] = i
* answer: f[a.length()][b.length()]

{% highlight C++ %}
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
{% endhighlight %}
压缩一维: 注意保存i-1 j-1
{% highlight C++ %}
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
{% endhighlight %}

### 3. [Interleaving String - Leetcode 97](https://leetcode.com/problems/interleaving-string/)
```
Given s1, s2, s3, find whether s3 is formed by the interleaving of s1 and s2.

For example,
Given:
s1 = "aabcc",
s2 = "dbbca",

When s3 = "aadbbcbcac", return true.
When s3 = "aadbbbaccc", return false.
```

* state: f[i][j]第一个字符串的前i个字符，匹配上第二个字符串的前j个字符，能否交错构成第三个字符串的前i+j个字符
* function: f[i][j] = f[i-1][j] (if a[i] == c[i+j]) or f[i][j-1] (if b[j] == c[i+j])
* intialize: f[i][0]={a的前i个字符==c的前i个字符}; f[0][i]=同理可得
* answer: f[a.length()][b.length()]

{% highlight C++ %}
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
{% endhighlight %}
压缩：稍微复杂，以后再说，网上有答案

### 4. [Distinct Subsequence - Leetcode115 Hard](https://leetcode.com/problems/distinct-subsequences/)
```
Given a string S and a string T, count the number of distinct subsequences of T in S.

A subsequence of a string is a new string which is formed from the original string by deleting some (can be none) of the characters without disturbing the relative positions of the remaining characters. (ie, "ACE" is a subsequence of "ABCDE" while "AEC" is not).

Here is an example:
S = "rabbbit", T = "rabbit"

Return 3.
```

* state: f[i][j]第一个字符串的前i个字符，匹配上第二个字符串的前j个字符，有多少个distinct subseqs
* function: f[i][j] = f[i-1][j] + f[i-1][j-1] (if a[i] == b[j]) or f[i-1][j] (if a[i] != b[j],没有删除j的这种情况了)
* intialize: f[i][0]=1, f[0][i] = 0 (i>0)
* answer: f[a.length()][b.length()]

{% highlight C++ %}
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
{% endhighlight %}
压缩一维： 不太好理解
{% highlight C++ %}
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
{% endhighlight %}
