---
layout: post
title:  "Basics on Dynamic Programming(DP) 6 - Word Break"
date:   2016-03-20 15:30:00
tags: [algorithm, leetcode, dynamic programming, dp, word break]
categories: Algorithm
---

> 字符串处理，Word Break

> 一般有N个数/字符，就开N+1个位置的数组，第0个位置单独留出来作初始化

### 1. [Word Break - Leetcode 139](https://leetcode.com/problems/word-break/)
```
Given a string s and a dictionary of words dict, determine if s can be segmented into a space-separated sequence of one or more dictionary words.

For example, given
s = "leetcode",
dict = ["leet", "code"].

Return true because "leetcode" can be segmented as "leet code".
```
{% highlight C++ %}
bool wordBreak(string s, unordered_set<string> &dict) {
  int wordLength = 0;
  for (auto word : dict) {
    wordLength = max(wordLength, (int)word.size());
  }
  // DP[i]代表，s[0,i) 前i个字符是否能满足答案
  // 方程dp[i] = dp[j] && j+1~i属于dict
  int n = s.size();
  vector<bool> dp(n + 1, false);  //初始化，长度为 n 的字符串有 n+1 个隔板
  dp[0] = true;  //这样是为了减少0位置的判断,空字符串
  for (int i = 1; i <= n; i++) {
    // 超时了，优化: 切分位置的枚举，从后往前，到单词的最长位置
    /*for (int j = 0; j < i; j++) {
      if (dp[j] && dict.find(s.substr(j, i - j)) != dict.end()) {  //从j开始算的
        dp[i] = true;
        break;
      }
    }*/
    for (int j = 1; j <= wordLength && j <= i; j++) {
      if (dp[i - j] && dict.find(s.substr(i - j, j)) != dict.end()) {  //从j开始算的
        dp[i] = true;
        break;
      }
    }
  }
  return dp[n];
}
{% endhighlight %}

### 2. [Word Break II- Leetcode 140](https://leetcode.com/problems/word-break-ii/)
```
Given a string s and a dictionary of words dict, add spaces in s to construct a sentence where each word is a valid dictionary word.

Return all such possible sentences. ==>这种题目，不能用DP，只能DFS

For example, given
s = "catsanddog",
dict = ["cat", "cats", "and", "sand", "dog"].

A solution is ["cats and dog", "cat sand dog"].
```
DFS原始代码会超时：
{% highlight C++ %}
vector<string> wordBreak(string s, unordered_set<string> &dict) {
  vector<string> res;
  DFS(s, 0, dict, "", res);
  return res;
}
void DFS(string s, int index, unordered_set<string> &dict, string path,
         vector<string> &res) {
  if (index == s.size()) {
    res.push_back(path);
    return;
  }
  for (int i = index; i < s.size(); i++) {
    if (dict.find(s.substr(index, i - index + 1)) != dict.end()) {
      string tmp_path = path;  // tmp_path不改变原始path
      tmp_path += (path == "") ? "" : " ";  //是否有空格
      tmp_path += s.substr(index, i - index + 1);
      DFS(s, i + 1, dict, tmp_path, res);
    }
  }
}
{% endhighlight %}
优化：重复计算太多，可以借鉴DP，存储所有的合法的单词 - 难!!
{% highlight C++ %}
vector<string> wordBreak(string s, unordered_set<string> &dict) {
  // 先用1的方法得到结果的组数，再分别求结果
  // prev[i][j]表示s[i, j)是一个合法单词,可以从j处切开
  vector<vector<bool> > prev(s.length(), vector<bool>(s.length() + 1, false));
  vector<bool> dp(s.size() + 1, false);  //初始化为false!
  dp[0] = true;                          // 空字符串
  for (int i = 1; i <= s.size(); i++)
    for (int j = i - 1; j >= 0; j--) {
      if (dp[j] && dict.find(s.substr(j, i - j)) != dict.end()) {
        dp[i] = true;
        prev[j][i] = true;
      }
    }
  vector<string> result;
  if (!dp[s.size()]) return result;
  gen_path(s, prev, 0, "", result);
  reverse(result.begin(), result.end());
  return result;
}
// DFS遍历树，生成路径
void gen_path(string s, const vector<vector<bool> > &prev, int cur, string path,
              vector<string> &result) {
  if (cur == s.size()) {  //找到，存入
    result.push_back(path);
    return;  // string path恢复原状
  }
  for (int i = cur + 1; i <= s.size(); i++) {  //找从cur开始的，可行的解!
    if (prev[cur][i]) {
      path += (path == "") ? "" : " ";  //是否有空格
      path += s.substr(cur, i - cur);
      gen_path(s, prev, i, path, result);  //这里还是i!
      path = path.substr(0, path.size()-i+cur);
      if(path != "")  path = path.substr(0, path.size()-1);
    }
  }
}
{% endhighlight %}