---
layout: post
title:  "Combination and Permutation 2 - General Template"
date:   2016-03-22 22:30:00
tags: [algorithm, leetcode, DFS, combination, permutation]
categories: Algorithm
---

### 1. [Letter Combinations of a Phone Number - Leetcode 17](https://leetcode.com/problems/letter-combinations-of-a-phone-number/)
![phonenumber](http://7xno5y.com1.z0.glb.clouddn.com/phonenumber.png)
```
Given a digit string, return all possible letter combinations that the number could represent.
A mapping of digit to letters (just like on the telephone buttons) is given below.

Input:Digit string "23"
Output: ["ad", "ae", "af", "bd", "be", "bf", "cd", "ce", "cf"].
Note:
Although the above answer is in lexicographical order, your answer could be in any order you want.
```

利用通用模板:
{% highlight C++ %}
vector<string> letterCombinations(string digits) {
        //DFS,深搜解法!!
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
        //处理start  这一部分，和模板有些许区别。循环处理start中代表的各个字母。本题要求是，每个数字确定了，且按顺序，所以与正常的combination不同
        for(int i=0; i<hash[digits[start]-'0'].size(); i++) {
            path += hash[digits[start]-'0'][i];
            letterCombinationsCore(digits, res, path, hash, start+1);
            path.erase(path.size()-1, 1);   //从a位置开始，erase b个字符
        }
}
{% endhighlight %}
增量构造法，必须注意res的初始化条件，一开始的时候必须有至少一个元素，然后遍历res里的所有元素，每个添加新的元素, 这个不好：
{% highlight C++ %}
vector<string> letterCombinations(string digits) {
        //0 and 1?貌似不用管  creat hash first
        string hash[10] = {" ", "'", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"};
        //迭代法就OK  但是res里，必须是有至少1个值的，不能从0开始
        int times = digits.size();
        vector<string> res;
        if(times == 0) {    //应该返回[""]!!
            res.push_back("");
            return res;
        } else {
            for(int i=0; i<hash[digits[0]-'0'].size(); i++)
                res.push_back(string("") + hash[digits[0]-'0'][i]);  //这么变成string!!  别忘了digits[0] - '0'
        }
        for(int i=1; i<times; i++) {    //还剩几个数字没处理
            vector<string> copyRes(res);
            int num = res.size();
            res.clear();    //每次都，只存入最新的!!
            for(int k=0; k<num; k++) //循环之前的每一个
                for(int j=0; j<hash[digits[i]-'0'].size(); j++) {   //每一个新的数字，代表的每个字母加进去
                    res.push_back(copyRes[k] + hash[digits[i]-'0'][j]);  //简洁一些string path(res[k]); path += hash[digits[i]-'0'][j];
                }
        }
        return res;
}
{% endhighlight %}
删掉了初始化的条件，简洁代码：时间复杂度 O(3^n)，空间复杂度O(1),BFS广搜解法!!
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
                res.push_back(copyRes[k] + hash[digits[i]-'0'][j]);  //简洁一些string path(res[k]); path += hash[digits[i]-'0'][j];
            }
    }
    return res;
}
{% endhighlight %}

5，Combination Sum
Given a set of candidate numbers (C) and a target number (T), find all unique combinations in C where the candidate numbers sums to T.
The same repeated number may be chosen from C unlimited number of times.
Note:
All numbers (including target) will be positive integers.
Elements in a combination (a1, a2, … , ak) must be in non-descending order. (ie, a1 ≤ a2 ≤ … ≤ ak).
The solution set must not contain duplicate combinations.
For example, given candidate set 2,3,6,7 and target 7, 
A solution set is: 
[7] 
[2, 2, 3]
利用通用模板 2
vector<vector<int> > combinationSum(vector<int> &candidates, int target) {
        //DFS 深搜
        sort(candidates.begin(), candidates.end());
        vector<vector<int> > res;
        vector<int> path;
        combinationSumCore(candidates, res, path, target, 0, 0);
        return res;  //!!别忘返回
}
void combinationSumCore(vector<int> &candidates, vector<vector<int> > &res, vector<int> &path, int target, int start, int sum) {
        //if(sum > target)  加入优化后，不需要了！
        //    return;
        if(sum == target){
            res.push_back(path);
            return;
        }
        for(int i=start; i<candidates.size(); i++) {
            sum += candidates[i];
            if(sum > target)    break;  //优化：说明不用再向后走了，已经超了，不可能加入candidates[i]能实现
            path.push_back(candidates[i]);
            combinationSumCore(candidates, res, path, target, i, sum);   //also same "i", because same repeated number is allowed
            path.pop_back();
            sum -= candidates[i];
        }
}

6，Combination Sum II
Given a collection of candidate numbers (C) and a target number (T), find all unique combinations in C where the candidate numbers sums to T.
Each number in C may only be used once in the combination.
Note:
All numbers (including target) will be positive integers.
Elements in a combination (a1, a2, … , ak) must be in non-descending order. (ie, a1 ≤ a2 ≤ … ≤ ak).
The solution set must not contain duplicate combinations.
For example, given candidate set 10,1,2,7,6,1,5 and target 8, 
A solution set is: 
[1, 7] 
[1, 2, 5] 
[2, 6] 
[1, 1, 6]
与上题区别，一个数字不能用多次，那就改Core中递归调用那一行就OK
另：不能有duplicate的解：
void combinationSum2Core(vector<int> &candidates, vector<vector<int> > &res, vector<int> &path, int target, int start, int sum) {
        if(sum == target){
            if(find(res.begin(), res.end(), path) == res.end())
                res.push_back(path);    //[1,1], 1 Output:    [[1],[1]] Expected:    [[1]]
            return;
        }
        for(int i=start; i<candidates.size(); i++) {
            sum += candidates[i];
            if(sum > target)    break;  //优化：说明不用再向后走了，已经超了，不可能加入candidates[i]能实现
            path.push_back(candidates[i]);
            combinationSum2Core(candidates, res, path, target, i+1, sum);   //also same "i", because same repeated number is allowed
            path.pop_back();
            sum -= candidates[i];
        }
}
更好的解决方案： 就跟subset II中的解决方案一样
void combinationSum2Core(vector<int> &candidates, vector<vector<int> > &res, vector<int> &path, int target, int start, int sum) {
        if(sum == target){
            res.push_back(path);
            return;
        }
        for(int i=start; i<candidates.size(); i++) {
            if(i != start && candidates[i] == candidates[i-1])    
                continue;
            sum += candidates[i];
            if(sum > target)    break;  //优化：说明不用再向后走了，已经超了，不可能加入candidates[i]能实现
            path.push_back(candidates[i]);
            combinationSum2Core(candidates, res, path, target, i+1, sum);   //also same "i", because same repeated number is allowed
            path.pop_back();
            sum -= candidates[i];
        }
}

Combination Sum III  216  (https://leetcode.com/problems/combination-sum-iii/)

Find all possible combinations of k numbers that add up to a number n, given that only numbers from 1 to 9 can be used and each combination should be a unique set of numbers.

Ensure that numbers within the set are sorted in ascending order.


Example 1:

Input: k = 3, n = 7

Output:

[[1,2,4]]

Example 2:

Input: k = 3, n = 9

Output:

[[1,2,6], [1,3,5], [2,3,4]]

public List<List<Integer>> combinationSum3(int k, int n) {
    List<List<Integer>> res = new ArrayList<List<Integer>>();
    List<Integer> path = new ArrayList<Integer>();
    DFS(res, k, n, 0, path, 0);
    return res;
}
void DFS(List<List<Integer>> res, int k, int n, int step, List<Integer> path, int sum) {
    if(path.size() == k) {
        if(sum == n)  res.add(new ArrayList(path));
        return;
    }
    for(int i=step+1; i<=9; i++) {
        path.add(i);
        DFS(res, k, n, i, path, sum+i);
        path.remove(path.size() - 1);
    }
}

7，Permutations
Given a collection of numbers, return all possible permutations.
For example,
[1,2,3] have the following permutations:
[1,2,3], [1,3,2], [2,1,3], [2,3,1], [3,1,2], and [3,2,1].

通用模板：与组合区别，需要加个标记数组（或者每次push前，find一下path里是否已经有了），同时不需要start参数，Core中循环每次从0开始
时间复杂度 O(n!)，空间复杂度 O(n)  1
vector<vector<int> > permute(vector<int> &num) {
        vector<vector<int> > res;
        vector<int> path;
        vector<bool> visited(num.size(), false);    //与组合区别:需要有标记数组;core中传入的参数不用start了
        permuteCore(res, path, num, visited);
        return res;
}
void permuteCore(vector<vector<int> > &res, vector<int> &path, vector<int> &num, vector<bool> &visited) {
        if(path.size() == num.size()) {  //all numbers
            res.push_back(path);
            return;
        }
        for(int i=0; i<num.size(); i++) {
            if(visited[i])  //访问过的，不能再访问了；这里貌似可以直接利用path.find(element)是否包含
                continue;
            path.push_back(num[i]);
            visited[i] = true;
            permuteCore(res, path, num, visited);
            path.pop_back();
            visited[i] = false;
     }
}
其他方法：
1，swap方法 2
vector<vector<int> > permute(vector<int> &num) {
        vector<vector<int> > res;
        permuteCore(res, num, 0);   //num就相当于path了
        return res;
}
void permuteCore(vector<vector<int> > &res, vector<int> &num, int start) {
        if(start == num.size()) {
            res.push_back(num);
            return;
        }
        for(int i=start; i<num.size(); i++) {
            swap(num[start], num[i]);   // 全排列的主要思路就是交换数组的元素，在递归后需要交换回来。注意循环前要首先递归一次!!!因为有不换的情况
            permuteCore(res, num, start+1); //换了后，求start后边开始的结果
            swap(num[start], num[i]);   
        }
}
2, STL法 2
vector<vector<int> > permute(vector<int> &num) {
    //利用STL 中std::next_permutation()，时间复杂度 O(n!)，空间复杂度 O(1)
    vector<vector<int> > res;
    sort(num.begin(), num.end());
    do {
        res.push_back(num);
    }while(next_permutation(num.begin(), num.end()));
    return res;
}
3, 插入法  迭代：
vector<vector<int> > permute(vector<int> &num) {
    //插入法，迭代
    vector<vector<int> > res(1);    //need one 初始值
    for(int i=0; i<num.size(); i++) {   //for each num, insert it into...
        vector<vector<int> > copyRes(res);  //这种只要最后的结果的，都需要copyRes来赋值一份，跟上边求几个组合相同
        res.clear();
        for(int j=0; j<copyRes.size(); j++) {  //for each res, insert the i
            for(int k=0; k<=copyRes[j].size(); k++) {   //there are res[j].size() places to insert
                vector<int> line(copyRes[j]);
                line.insert(line.begin()+k, num[i]);
                res.push_back(line);
            }
        }
    }
    return res;
}

8，Permutations II
Given a collection of numbers that might contain duplicates, return all possible unique permutations.
For example,
[1,1,2] have the following unique permutations:
[1,1,2], [1,2,1], and [2,1,1]
通用模板：与7区别，就是排序，且有continue中的判断。
vector<vector<int> > permuteUnique(vector<int> &num) {
        vector<vector<int> > res;
        vector<int> path;
        vector<bool> visited(num.size(), false);    //与组合区别:需要有标记数组;core中传入的参数不用start了
        sort(num.begin(), num.end());   //这里需要排序!
        permuteCore(res, path, num, visited);
        return res;
}
void permuteCore(vector<vector<int> > &res, vector<int> &path, vector<int> &num, vector<bool> &visited) {
        if(path.size() == num.size()) {  //all numbers
            res.push_back(path);
            return;
        }
        for(int i=0; i<num.size(); i++) {
            if(visited[i] || (i!=0 && num[i]==num[i-1] && visited[i-1]))  
            //访问过的，和上一个相同的 且上一个访问过的，都不能再访问了==>只有这儿不同； 如1,1这样的，只有第2个1进入，然后第一个1才能入，顺序的
                continue;
            path.push_back(num[i]);
            visited[i] = true;
            permuteCore(res, path, num, visited);
            path.pop_back();
            visited[i] = false;  
        }
}
其他方法：迭代+判重
vector<vector<int> > permuteUnique(vector<int> &num) {
        //插入法，迭代 ==> 判断重复
        vector<vector<int> > res(1);    //need one 初始值
        for(int i=0; i<num.size(); i++) {   //for each num, insert it into...
            int resNum = res.size();
            vector<vector<int> > copyRes(res);  //这种只要最后的结果的，都需要copyRes来赋值一份，跟上边求几个组合相同
            res.clear();
            for(int j=0; j<resNum; j++) {  //for each res, insert the i
                for(int k=0; k<=copyRes[j].size(); k++) {   //there are res[j].size() places to insert
                    vector<int> line(copyRes[j]);
                    line.insert(line.begin()+k, num[i]);
                    if(find(res.begin(), res.end(), line) == res.end())
                        res.push_back(line);
                    while(k < line.size()-1 && num[i] == line[k+1])  k++;    //过滤重复的位置==>必须有这句优化，否则超时！！！
                }
            }
        }
        return res;
}

9，Palindrome Partitioning
Given a string s, partition s such that every substring of the partition is a palindrome.
Return all possible palindrome partitioning of s. 这种的不是DP，就是DFS
For example, given s = "aab",
Return
  [
    ["aa","b"],
    ["a","a","b"]
  ]
通用模板：
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

10，Next Permutation
Implement next permutation, which rearranges numbers into the lexicographically next greater permutation of numbers.
If such arrangement is not possible, it must rearrange it as the lowest possible order (ie, sorted in ascending order).
The replacement must be in-place, do not allocate extra memory.
Here are some examples. Inputs are in the left-hand column and its corresponding outputs are in the right-hand column.
1,2,3 → 1,3,2
3,2,1 → 1,2,3
1,1,5 → 1,5,1


void nextPermutation(vector<int> &num) {
    int index1 = num.size()-1, index2 = num.size()-1;
    while(index1>0 && num[index1]<=num[index1-1]) {
        index1--;
    }
    if(index1 > 0) {
        while(index2>=index1 && num[index2]<=num[index1-1]){    //注意相同的情况1,1,5-> 5,1,1
            index2--;
        }
        swap(num[index1-1], num[index2]);
    }
    reverse(num.begin()+index1, num.end());
}