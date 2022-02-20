---
layout: post
title:  "Combination and Permutation 4 - Real Problem"
date:   2016-03-23 10:30:00
tags: [algorithm, leetcode, DFS, combination, permutation]
categories: Algorithm
---

### 1. [Letter Combinations of a Phone Number - Leetcode 17](https://leetcode.com/problems/letter-combinations-of-a-phone-number/)
![phonenumber](/res/phonenumber.png)

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
  if (digits.empty()) return vector<string>();
  const string hash[10] = {" ",   "'",   "abc",  "def", "ghi",
                           "jkl", "mno", "pqrs", "tuv", "wxyz"};
  vector<string> res;
  letterCombinationsCore(digits, res, "", hash, 0);
  return res;
}
void letterCombinationsCore(string digits, vector<string> &res, string path,
                            const string *hash, int start) {
  if (start == digits.size()) {  //收敛条件，处理完了
    res.push_back(path);
    return;
  }
  //处理start，这里和模板有些许区别。循环处理start中代表的各个字母
  for (int i = 0; i < hash[digits[start] - '0'].size(); i++) {
    letterCombinationsCore(digits, res, path + hash[digits[start] - '0'][i],
                           hash, start + 1);
  }
}
{% endhighlight %}
增量构造法，简洁代码：时间复杂度O(n^3)，空间复杂度O(1),BFS广搜解法
{% highlight C++ %}
vector<string> letterCombinations(string digits) {
  if (digits.empty()) return vector<string>();
  const string hash[10] = {" ",   "'",   "abc",  "def", "ghi",
                           "jkl", "mno", "pqrs", "tuv", "wxyz"};
  //迭代法就OK  但是res里，必须是有至少1个值的，不能从0开始
  vector<string> res(1, "");                 //若times=0, 应该返回[""]!!
  for (int i = 0; i < digits.size(); i++) {  //还剩几个数字没处理
    vector<string> copyRes(res);
    res.clear();                              //每次都，只存入最新的!!
    for (int k = 0; k < copyRes.size(); k++)  //循环之前的每一个
      //每一个新的数字，代表的每个字母加进去
      for (int j = 0; j < hash[digits[i] - '0'].size(); j++)
        res.push_back(copyRes[k] + hash[digits[i]-'0'][j]);
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
//一个长度为 n 的字符串，有n-1个地方可以砍断，每个地方可断可不断，因此复杂度为O(2^n)
vector<vector<string>> partition(string s) {
  vector<vector<string>> res;
  vector<string> path;
  partitionCore(res, path, s, 0);
  return res;
}
void partitionCore(vector<vector<string>> &res, vector<string> &path, string s,
                   int start) {
  if (start == s.size()) {
    res.push_back(path);
    return;
  }
  for (int i = start; i < s.size(); i++) {  //从start开始往后切
    if (isPalindrome(s.substr(start, i - start + 1))) {
      path.push_back(s.substr(start, i - start + 1));
      partitionCore(res, path, s, i + 1);
      path.pop_back();
    }
  }
}
// 可以优化加速：借鉴DP思想，用二维数组存i到j是否为回文
bool isPalindrome(string s) {
  if (s.size() <= 1) return true;
  int i = 0, j = s.size() - 1;
  while (i <= j) {
    if (s[i] != s[j]) return false;
        i++;    j--;
    }
    return true;
}
{% endhighlight %}

### 3. [N Queens](https://www.lintcode.com/problem/33/)
```
Placing n queens on an n×n chessboard, and the queens can't be in the same row, column, diagonal line.Given an integer n, return all distinct solutions.
Each solution contains a distinct board configuration of the N-queens' placement, where 'Q' and '.' each indicate a queen and an empty space respectively.
```
return all distinct solutions -> DFS：
{% highlight C++ %}
void printQ(vector<vector<string>> &results, vector<vector<int>> &positions) {
    for(auto pos: positions) {
        vector<string> res;
        for(int i = 0; i < pos.size(); i++) {
            // string line {};
            // for(int j = 0; j < pos.size(); j++) {
            //     if(j == pos[i]) line += "Q";
            //     else line += ".";
            // }
            string line(pos.size(), '.');  // 注意string的这种初始化方式，类似vector!
            line[pos[i]] = 'Q';
            res.push_back(line);
        }
        results.push_back(res);
    }
}
bool canput(vector<int> &path, int pos) {
    for(int i = 0; i < path.size(); i++) {
        if(pos == path[i] || abs(pos - path[i]) == path.size() - i) {
            return false;
        }
    }
    return true;
}
void dfs(vector<vector<int>> &positions, vector<int> &path, int row, int N) {
    if(row == N) {
        positions.push_back(path);
        return;
    }
    // 第row行，从0位置开始放
    for(int i = 0; i < N; i++) {
        if(!canput(path, i)) continue;  // 这种continue方式代码缩进更少
        path.push_back(i);
        dfs(positions, path, row + 1, N);
        path.pop_back();
    }
}
vector<vector<string>> solveNQueens(int n) {
    // DFS
    vector<vector<int>> positions;
    vector<int> path;
    dfs(positions, path, 0, n);  // 从第0行开始
    vector<vector<string>> result;
    printQ(result, positions);
    return result;
}
{% endhighlight %}

### 4. [Word Ladder](https://www.lintcode.com/problem/120/)
```
find the shortest transformation sequence from start to end, output the length of the sequence
```
find the shortest path -> BFS：
{% highlight C++ %}
int ladderLength(string &start, string &end, unordered_set<string> &dict) {
    // 最短长度，那就是BFS啦
    unordered_set<string> visited;
    queue<string> q;
    q.push(start);
    visited.insert(start);
    int shortest = 2;
    while(!q.empty()) {
        // 把上一轮的都pop出来
        for(int i = q.size(); i > 0; i--) {
            string cur = q.front();
            q.pop();
            for(char c = 'a'; c <= 'z'; c++) {
                for(int l = 0; l < cur.size(); l++) {
                    string nextcur = cur;
                    nextcur[l] = c;
                    // 先判断，如果对了，就退出了。因为end不在dict里
                    if(nextcur == end) return shortest;
                    // 是它自己，或者dict里没有，就continue
                    if(nextcur == cur || dict.find(nextcur) == dict.end()) continue;
                    if(visited.find(nextcur) != visited.end()) continue;  // !!别忘了，如果放过了，否则死循环
                    q.push(nextcur);
                    visited.insert(nextcur);
                }
            }
        }
        shortest++;
    }
    return 0;
}
// 拓展：找出所有从start到end的最短转换序列
// 那么需要在BFS过程中，记录图的边map<string, vector<string>> next_word_dict;然后用DFS找distance为最短的path
// 剪枝：用距离，比如存unordered_map<string, int> distance(代替之前的visited)，保证distance是一个越来越近的状态
{% endhighlight %}
