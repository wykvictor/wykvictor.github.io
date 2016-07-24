---
layout: post
title:  "Array & Numbers 3 - Sorted Array"
date:   2016-03-29 10:30:00
tags: [algorithm, leetcode, array, sorted array]
categories: Algorithm
---

### 1. [Merge k Sorted Arrays](http://www.lintcode.com/en/problem/merge-k-sorted-arrays/)
```
Do it in O(N log k).
N is the total number of integers.
k is the number of arrays.
```

priority_queue的使用
{% highlight C++ %}
struct Node {
  int val;
  int x, y;  // 坐标
  Node(int a, int b, int c) : val(a), x(b), y(c) {}
};

struct cmp {
  bool operator()(Node a, Node b) { return a.val > b.val; }
};

vector<int> mergekSortedArrays(vector<vector<int>>& arrays) {
  vector<int> res;
  if (arrays.empty()) return res;
  priority_queue<Node, vector<Node>, cmp> q;
  for (int i = 0; i < arrays.size(); i++) {
    if (!arrays[i].empty()) {
      q.push(Node(arrays[i][0], i, 0));
    }
  }
  while (!q.empty()) {
    Node tmp = q.top();
    q.pop();
    res.push_back(tmp.val);
    if (tmp.y + 1 < arrays[tmp.x].size()) {  // !! +1, 判断放不放下一个
      q.push(Node(arrays[tmp.x][tmp.y + 1], tmp.x, tmp.y + 1));
    }
  }
  return res;
}
{% endhighlight %}
