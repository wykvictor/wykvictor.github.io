---
layout: post
title:  "Data Structure 3 - Others"
date:   2016-05-28 18:30:00
tags: [algorithm, leetcode, data structure]
categories: Algorithm
---

### 1. [LRU Cache](http://www.lintcode.com/en/problem/lru-cache/)
```
Design and implement a data structure for Least Recently Used (LRU) cache. It should support the following operations: get and set.

get(key) - Get the value (will always be positive) of the key if the key exists in the cache, otherwise return -1.
set(key, value) - Set or insert the value if the key is not already present. When the cache reached its capacity, it should invalidate the least recently used item before inserting a new item.

2, [set(2,1),set(1,1),get(2),set(4,1),get(1),get(2)] --> [1,-1,1]
```

需要支持在中部，头部删除，尾部添加的数据结构
{% highlight C++ %}
#include <list>
class CacheNode {
 public:
  CacheNode(int k, int v) : key(k), value(v) {}
  int key;
  int value;
};
class LRUCache {
 public:
  LRUCache(int capacity) { this->capacity = capacity; }

  int get(int key) {
    if (cacheMap.find(key) == cacheMap.end()) {
      return -1;
    }
    // c1.splice(c1.beg,c2,c2.beg), list特有操作
    // 将c2的beg位置的元素连接到c1的beg位置，并且在c2中施放掉beg位置的元素
    cacheList.splice(cacheList.begin(), cacheList, cacheMap[key]);
    cacheMap[key] = cacheList.begin();  // list的iterator在insert后不change
    return cacheMap[key]->value;
  }

  void set(int key, int value) {
    if (cacheMap.find(key) == cacheMap.end()) {
      if (cacheList.size() == capacity) {
        cacheMap.erase(cacheList.back().key);  // front(), back()
        cacheList.pop_back();                  // pop_back弹出最后一个
      }
      cacheList.push_front(CacheNode(key, value));  // push_front操作
      cacheMap[key] = cacheList.begin();
      return;
    }
    // exist, move to begin
    cacheList.splice(cacheList.begin(), cacheList, cacheMap[key]);
    cacheMap[key] = cacheList.begin();  // list的iterator在insert后不change
    cacheMap[key]->value = value;
  }

 private:
  list<CacheNode> cacheList;
  unordered_map<int, list<CacheNode>::iterator> cacheMap;
  int capacity;
};

// 用list的简单的接口
class LRUCache {
public:
    LRUCache(int capacity) : cap(capacity) {
        // do intialization if necessary
    }
    int get(int key) {
        if(key2node.find(key) == key2node.end()) return -1;
        // 迁移数据到头部
        CacheNode curnode(*key2node[key]);
        nodelist.erase(key2node[key]);
        nodelist.push_front(curnode);
        key2node[key] = nodelist.begin();  // 更新索引, erase不会导致之前的iter失效
        return curnode.val;
    }
    void set(int key, int value) {
        // 如果有这个元素，直接赋值，并调到头部
        if(get(key) != -1) {  // 相似代码，不用再写了
            nodelist.front().val = value;  // 更新value
            return;
        }
        //如果没这个元素，就需要插入，先判断空间够不够
        if(key2node.find(key) == key2node.end()) {
            if(nodelist.size() >= this->cap) {
                key2node.erase(nodelist.back().key);
                nodelist.pop_back();
            }
        }
        nodelist.emplace_front(key, value);  // 用emplace
        key2node[key] = nodelist.begin();
    }
private:
    int cap;
    // list，支持从中取元素，放到头部
    list<CacheNode> nodelist;
    // hash，支持快速找到元素
    unordered_map<int, list<CacheNode>::iterator> key2node;
};
{% endhighlight %}

### 2. [Anagrams](http://www.lintcode.com/en/problem/anagrams/)
```
Given an array of strings, return all groups of strings that are anagrams. 
Two strings are anagram if they can be the same after change the order of characters.
Given ["lint", "intl", "inlt", "code"], return ["lint", "inlt", "intl"].
Given ["ab", "ba", "cd", "dc", "e"], return ["ab", "ba", "cd", "dc"].
```

传统方法, 最差O(n^2)
{% highlight C++ %}
// 传统方法，超时；空间少
bool isAnagrams(string src, string dst) {
  if (src.size() != dst.size()) return false;
  unordered_map<char, int> hash;
  for (int i = 0; i < src.size(); i++) {
    if (hash.find(src[i]) == hash.end()) {
      hash[src[i]] = 1;  // !! 出现过1次了，不是0
    } else {
      hash[src[i]]++;
    }
  }
  for (int i = 0; i < dst.size(); i++) {
    if (hash.find(dst[i]) == hash.end()) {
      return false;
    } else {
      hash[dst[i]]--;
    }
  }
  for (auto h : hash) {
    if (h.second != 0) return false;
  }
  return true;
}
// better solution
bool isAnagrams(string src, string dst) {
  if (src.size() != dst.size()) return false;  // 必须有
  int hash[256] = {0};  // 只需256byte，空间也没大多少
  for (int i = 0; i < src.size(); i++) {
    hash[src[i]]++;
  }
  for (int i = 0; i < dst.size(); i++) {
    hash[dst[i]]--;
    if (hash[dst[i]] < 0) return false;  // 巧妙！
  }
  return true;
}

vector<string> anagrams(vector<string>& strs) {
  vector<string> res;
  vector<bool> done(strs.size(), false);
  for (int i = 0; i < strs.size() - 1; i++) {
    if (done[i]) continue;
    done[i] = true;
    bool isres = false;
    for (int j = i + 1; j < strs.size(); j++) {
      if (isAnagrams(strs[i], strs[j])) {
        done[j] = true;
        res.push_back(strs[j]);
        isres = true;
      }
    }
    if (isres) res.push_back(strs[i]);
  }
  return res;
}
{% endhighlight %}
优化方法, O(n)
{% highlight C++ %}
// 桶排序，常数时间
string getSortedString(string &str) {
  int count[26] = {0};
  for (int i = 0; i < str.length(); i++) {
    count[str[i] - 'a']++;
  }
  string sorted_str = "";
  for (int i = 0; i < 26; i++) {
    for (int j = 0; j < count[i]; j++) {
      sorted_str = sorted_str + (char)('a' + i);
    }
  }
  return sorted_str;
}

vector<string> anagrams(vector<string> &strs) {
  unordered_map<string, int> hash;
  for (int i = 0; i < strs.size(); i++) {
    string str = getSortedString(strs[i]);
    // string str = strs[i];
    // sort(str.begin(), str.end());  // 或者就直接调用排序就可以，如果string平均长度都小于26
    if (hash.find(str) == hash.end()) {
      hash[str] = 1;
    } else {
      hash[str] = hash[str] + 1;
    }
  }
  // O(nL)存入，O(nL)查询
  vector<string> result;
  for (int i = 0; i < strs.size(); i++) {
    string str = getSortedString(strs[i]);
    if (hash.find(str) == hash.end()) continue;
    if (hash[str] > 1) {
      result.push_back(strs[i]);
    }
  }
  return result;
}
{% endhighlight %}

### 3. [Longest Consecutive Sequence](http://www.lintcode.com/en/problem/longest-consecutive-sequence/)
```
Given [100, 4, 200, 1, 3, 2],
The longest consecutive elements sequence is [1, 2, 3, 4]. Return its length: 4
Your algorithm should run in O(n) complexity.
```
{% highlight C++ %}
int longestConsecutive(vector<int> &num) {
    if(num.size() <= 1) return num.size();  // 特殊情况
    unordered_set<int> hash;  //int hash[10000];
    for(auto n: num) {
        hash.insert(n);
    }
    int res = 1;
    for(auto n: num) {
        int left = n - 1;
        while(hash.find(left) != hash.end()) {
            hash.erase(left);  // 这个很重要，否则超时!!算过了，后续碰到不需要再进来while了
            left--;
        }
        int right = n + 1;
        while(hash.find(right) != hash.end()) {
            hash.erase(right);
            right++;
        }
        res = max(res, right - left - 1);
    }
    return res;
}
{% endhighlight %}
