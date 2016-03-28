---
layout: post
title:  "Combination and Permutation 4 - Real Problem"
date:   2016-03-23 10:30:00
tags: [algorithm, leetcode, DFS, combination, permutation]
categories: Algorithm
---

### 1. [Letter Combinations of a Phone Number - Leetcode 17](https://leetcode.com/problems/letter-combinations-of-a-phone-number/)
![phonenumber](http://7xno5y.com1.z0.glb.clouddn.com/phonenumber.png)

```
Given a digit string, return all possible letter combinations that the number could represent.
A mapping of digit to letters (just like on the telephone buttons) is given as the above image.

Input:Digit string "23"
Output: ["ad", "ae", "af", "bd", "be", "bf", "cd", "ce", "cf"].
Note:
Although the above answer is in lexicographical order, your answer could be in any order you want.
```

利用通用模板:
{% highlight C++ %}
vector<string> letterCombinations(string digits) {
    // DFS,深搜解法!!
    const string hash[10] = {" ", "'", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"};
    vector<string> res;
    letterCombinationsCore(digits, res, "", hash, 0);
    return res;
}
void letterCombinationsCore(string digits, vector<string> &res, string path, const string *hash, int start) {
    if(start == digits.size()) {  //收敛条件，处理完了
        res.push_back(path);
        return;
    }
    //处理start，这里和模板有些许区别。循环处理start中代表的各个字母
    for(int i=0; i<hash[digits[start]-'0'].size(); i++) {
        path += hash[digits[start]-'0'][i];
        letterCombinationsCore(digits, res, path, hash, start+1);
        path.erase(path.size()-1, 1);   //从a位置开始，erase b个字符
    }
}
{% endhighlight %}
增量构造法，简洁代码：时间复杂度O(3^n)，空间复杂度O(1),BFS广搜解法!!
{% highlight C++ %}
vector<string> letterCombinations(string digits) {
    //0 and 1 貌似不用管  creat hash first  Use const is better!
    const string hash[10] = {" ", "'", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"};
    //迭代法就OK  但是res里，必须是有至少1个值的，不能从0开始
    vector<string> res(1, "");  //若times=0, 应该返回[""]!!
    for(int i=0; i<digits.size(); i++) {    //还剩几个数字没处理
        vector<string> copyRes(res);
        res.clear();    //每次都，只存入最新的!!
        for(int k=0; k<copyRes.size(); k++) //循环之前的每一个
            for(int j=0; j<hash[digits[i]-'0'].size(); j++) {   //每一个新的数字，代表的每个字母加进去
                res.push_back(copyRes[k] + hash[digits[i]-'0'][j]);
            }
    }
    return res;
}
{% endhighlight %}

### 2. [Palindrome Partitioning - Leetcode 131](https://leetcode.com/problems/palindrome-partitioning/)
```
Given a string s, partition s such that every substring of the partition is a palindrome.
Return all possible palindrome partitioning of s.  注意区别，不是DP ==> 是DFS
For example, given s = "aab",
Return
  [
    ["aa","b"],
    ["a","a","b"]
  ]
```

通用模板：
{% highlight C++ %}
//一个长度为 n 的字符串，有 n-1 个地方可以砍断，每个地方可断可不断，因此复杂度为O(2^n)
vector<vector<string>> partition(string s) {
    vector<vector<string>> res;
    vector<string> path;
    partitionCore(res, path, s, 0);
    return res;
}
void partitionCore(vector<vector<string>> &res, vector<string> &path, string s, int start) {
    if(start == s.size()) {
        res.push_back(path);
        return;
    }
    for(int i=start; i<s.size(); i++) {//从start开始往后切
        if(isPalindrome(s.substr(start, i-start+1))) {
            path.push_back(s.substr(start, i-start+1));
            partitionCore(res, path, s, i+1);
            path.pop_back();
        }
    }
}
bool isPalindrome(string s) {
    if(s.size() <= 1)
        return true;
    int i=0, j=s.size()-1;
    while(i <= j) {
        if(s[i] != s[j])
            return false;
        i++;    j--;
    }
    return true;
}
{% endhighlight %}
