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
{% endhighlight %}
