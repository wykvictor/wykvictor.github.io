---
layout: post
title:  "Data Structure 2 - Heap/TreeMap"
date:   2016-05-27 18:30:00
tags: [algorithm, leetcode, data structure, heap]
categories: Algorithm
---

> Heap

* Binary Tree, 完全二叉树
* 父节点 >= left/right，子节点间无特殊大小关系
* 插入节点：放到末尾，再往上调整
* 删除节点：首尾互换，再往下调整

### 1. [Heapify](http://www.lintcode.com/en/problem/heapify/)
```
Given an integer array, heapify it into a min-heap array.
For a heap array A, A[0] is the root of heap, and for each A[i], A[i * 2 + 1] is the left child of A[i] and A[i * 2 + 2] is the right child of A[i].

Given [3,2,1,4,5], return [1,2,3,4,5] or any legal heap array.
O(n) time complexity
```
{% highlight C++ %}
// Time: O(n)
void shiftDown(vector<int> &A, int cur) {
  int N = A.size();
  while (cur < N) {
    if (cur * 2 + 1 >= N) return;  // no child
    int replace;
    if (cur * 2 + 2 >= N) {  // only left child
      replace = cur * 2 + 1;
    } else {
      replace = A[cur * 2 + 2] > A[cur * 2 + 1] ? cur * 2 + 1 : cur * 2 + 2;
    }
    if (A[cur] <= A[replace]) return;
    swap(A[cur], A[replace]);
    cur = replace;
  }
}

void heapify(vector<int> &A) {
  int N = A.size();
  for (int i = N / 2; i >= 0; i--) {
    shiftDown(A, i);
  }
}
{% endhighlight %}

### 2. [Ugly Number II](http://www.lintcode.com/en/problem/heapify/)
```
Ugly number is a number that only have factors 2, 3 and 5.
Design an algorithm to find the nth ugly number. The first 10 ugly numbers are 1, 2, 3, 4, 5, 6, 8, 9, 10, 12...
If n=9, return 10.
Challenge: O(nlogn) or O(n) time
```
{% highlight C++ %}
// O(nlogn). 1, { 乘2, 3, 5 }
// 2, 3, 5 4, 6, 10 { 每次拿出堆里最小的元素，乘以2, 3, 5 }
// 需要很快的add, min == > pq 另：2 * 5, 5 * 2 = 10，用hash记录是否10已经放过了
int nthUglyNumber(int n) {
  if (n <= 1) return n;
  // greater是小顶堆，less是大顶堆，默认是less
  priority_queue<long long, vector<long long>, greater<long long>> pq;
  unordered_set<long long> s;
  pq.push(1);
  for (int i = 1; i < n; i++) {
    long long t = pq.top();
    pq.pop();
    if (s.find(2 * t) == s.end()) {
      s.insert(2 * t);
      pq.push(2 * t);
    }
    if (s.find(3 * t) == s.end()) {
      s.insert(3 * t);
      pq.push(3 * t);
    }
    if (s.find(5 * t) == s.end()) {
      s.insert(5 * t);
      pq.push(5 * t);
    }
  }
  return pq.top();
}
{% endhighlight %}
O(n)，模拟法
{% highlight C++ %}
// 用数组记录每一个丑数，一直算下一个，这样快，O(n) scan
// 空间换时间(相比与从1开始遍历并判断每一个数)
int nthUglyNumber(int n) {
  int index2 = 0, index3 = 0, index5 = 0;
  vector<int> uglys(n);
  uglys[0] = 1;
  for (int i = 1; i < n; i++) {
    uglys[i] =  // 取到下一个值是多少
        min(uglys[index2] * 2, min(uglys[index3] * 3, uglys[index5] * 5));
    while (uglys[index2] * 2 <= uglys[i]) {  // 注意是==!!!
      index2++;  // 一直找到乘以2能达到的最大的临界值
    }
    while (uglys[index3] * 3 <= uglys[i]) {
      index3++;  // 一直找到乘以2能达到的最大的临界值
    }
    while (uglys[index5] * 5 <= uglys[i]) {
      index5++;  // 一直找到乘以2能达到的最大的临界值
    }
  }
  return uglys[n - 1];
}
{% endhighlight %}

### 3. [Top k Largest Numbers II](http://www.lintcode.com/en/problem/top-k-largest-numbers-ii/)
```
Implement a data structure, provide two interfaces:
add(number). Add a new number in the data structure.
topk(). Return the top k largest numbers in this data structure.
k is given when we create the data structure.
```
{% highlight C++ %}
class Solution {
 public:
  //注意逻辑：用最小堆(从小到大)，和k个里边的最小的PK，一直保留k个
  priority_queue<int, vector<int>, greater<int>> pq;
  int capacity;
  Solution(int k) { capacity = k; }
  void add(int num) {
    if (pq.size() < capacity) {
      pq.push(num);
    } else {
      if (pq.top() < num) {
        pq.pop();
        pq.push(num);
      }
    }
  }
  vector<int> topk() {
    vector<int> res;
    priority_queue<int, vector<int>, greater<int>> pq_t(pq);
    // 注意判断pq.size()!!!
    int actualy_size = pq_t.size();
    for (int i = 0; i < actualy_size; i++) {
      res.push_back(pq_t.top());
      pq_t.pop();
    }
    reverse(res.begin(), res.end());
    return res;
  }
};
{% endhighlight %}

### 4. [Data Stream Median](https://www.lintcode.com/problem/81/)
```
Numbers keep coming, return the median of numbers at every time a new number added.
If there are n numbers in a sorted array A, the median is A[(n - 1) / 2]
For example, if A=[1,2,3], median is 2. If A=[1,19], median is 1.

Total run time in O(nlogn).
```
{% highlight C++ %}
class Solution {
public:
    void add(int val) {
        maxq.push(val);  // 先无脑放第1个里
        // balancing: 保持maxq比minq最多长1
        if(maxq.size() > minq.size() + 1) {
            minq.push(maxq.top());
            maxq.pop();
        }
        // balancing: 保持maxq的top()不大于minq.top()
        if(!minq.empty() && maxq.top() > minq.top()) {
            int tmp = minq.top();
            minq.pop();
            minq.push(maxq.top());
            maxq.pop();
            maxq.push(tmp);
        }
    }
    int getMedian() {
        if(maxq.empty()) return -1;
        return maxq.top();
    }
private:
    priority_queue<int, vector<int>, greater<int>> minq;
    priority_queue<int> maxq;
};
{% endhighlight %}

### 5. [Kth Smallest Number in Sorted Matrix](http://www.lintcode.com/en/problem/kth-smallest-number-in-sorted-matrix/)
```
Find the kth smallest number in at row and column sorted matrix.
Given k = 4 and a matrix:
[
  [1 ,5 ,7],
  [3 ,7 ,8],
  [4 ,8 ,9],
]
return 5

O(k log n), n is the maximal number in width and height.
```
{% highlight C++ %}
class Node {
 public:
  int val;
  int x;
  int y;
  Node(int v, int xx, int yy) : val(v), x(xx), y(yy) {}
  // 自定义Node，方法1，priority_queue<Node>
  // friend bool operator<(const Node& n1, const Node& n2) {
  //   return n1.val > n2.val;
  // }  // > is min heap

  // 自定义Node，方法2，priority_queue<Node>, const!
  // bool operator<(const Node& other) const {
  //   return val > other.val;
  // }  // > is min heap
};

// 自定义Node，方法3，priority_queue<Node, vector<Node>, cmp>
struct cmp {  // struct is Public in default!!
  bool operator()(const Node& n1, const Node& n2) { return n1.val > n2.val; }  // > is min heap
};

int kthSmallest(vector<vector<int>>& matrix, int k) {
  // priority_queue<Node> min_heap;  // call operator < to compare
  priority_queue<Node, vector<Node>, cmp> min_heap;  // 3 ways to use heap
  // 方法4
  // auto cmp = [](const Node &a, const Node &b) -> bool { return a.val > b.val; };
  // std::priority_queue<Node, vector<Node>, decltype(cmp)> pq(cmp);
  int M = matrix.size(), N = matrix[0].size();
  if (M * N < k) return -1;  // 極端情況
  vector<bool> visited(M * N, false);  // 一定注意，需要这个，否则一个点放进去2次
  min_heap.push(Node(matrix[0][0], 0, 0));
  visited[0] = true;

  for (int i = 1; i < k; i++) {
    Node cur = min_heap.top();
    min_heap.pop();
    // go down
    if (cur.x + 1 < M && !visited[(cur.x + 1) * N + cur.y]) {
      min_heap.push(Node(matrix[cur.x + 1][cur.y], cur.x + 1, cur.y));
      visited[(cur.x + 1) * N + cur.y] = true;
    }
    // go right
    if (cur.y + 1 < N && !visited[cur.x * N + cur.y + 1]) {
      min_heap.push(Node(matrix[cur.x][cur.y + 1], cur.x, cur.y + 1));
      visited[cur.x * N + cur.y + 1] = true;
    }
  }
  return min_heap.top().val;
}
{% endhighlight %}

### 6. [Top K Frequent Words](http://www.lintcode.com/en/problem/top-k-frequent-words/)
```
Given a list of words and an integer k, return the top k frequent words in the list.
Do it in O(nlogk) time and O(n) extra space.

Extra points if you can do it in O(n) time with O(k) extra space approximation algorithms.
```
{% highlight C++ %}
class Node {
 public:
  string key;
  int times;
  Node(string _key, int _times) : times(_times), key(_key) {}
  bool operator<(const Node& other) const {
    return times > other.times || (times == other.times && key < other.key);
  }
};

vector<string> topKFrequentWords(vector<string>& words, int k) {
  unordered_map<string, int> hash;
  for (auto a : words) {
    if (hash.find(a) == hash.end()) hash[a] = 0;
    hash[a]++;
  }
  priority_queue<Node> min_heap;  // 用小顶堆!!
  for (auto it: hash) {
    Node cur(it.first, it.second);
    min_heap.push(cur);
    // 不能直接利用重载的<比较，反着的if (min_heap.top() < cur) {
    if (min_heap.size() > k) {
      min_heap.pop();
    }
  }
  vector<string> res;
  while (!min_heap.empty()) {
    res.push_back(min_heap.top().key);
    min_heap.pop();
  }
  reverse(res.begin(), res.end());
  return res;
}
{% endhighlight %}

### 7. [High Five](http://www.lintcode.com/en/problem/high-five/)
```
There are two properties in the node student id and scores, 
to ensure that each student will have at least 5 points,
find the average of 5 highest scores for each person.
```
{% highlight C++ %}
double average(priority_queue<int, vector<int>, greater<int>>& pq) {
  double res = 0;
  int size = pq.size();
  while (!pq.empty()) {
    res += pq.top();
    pq.pop();
  }
  return res / size;
}
map<int, double> highFive(vector<Record>& results) {
    unordered_map<int, priority_queue<int, vector<int>, greater<int>>> infos;
    for (auto i : results) {
        // if(infos.find(i.id) == infos.end()) {
        //     // priority_queue<int, vector<int>, greater<int>> q;
        //     // q.push(i.score);
        //     // infos[i.id] = q;
        //     // 不需要这么写，https://www.cplusplus.com/reference/unordered_map/unordered_map/operator[]/
        //     infos[i.id].push(i.score);  // 构造参数，默认构造了空的priority_queue
        //     continue;
        // }
        if(infos[i.id].size() < 5) {
            infos[i.id].push(i.score);
        } else if(i.score > infos[i.id].top()) {
            infos[i.id].pop();
            infos[i.id].push(i.score);
        }
    }
    map<int, double> res;
    for(auto info: infos) {
        res[info.first] = average(info.second);
    }
    return res;
}
{% endhighlight %}

### 8.[K Closest Points](http://www.lintcode.com/en/problem/k-closest-points/)
{% highlight C++ %}
int dist(const Point& p1, const Point& p2) {
    return pow((p1.x - p2.x), 2) + pow((p1.y - p2.y), 2);
}
// 注意抽象函数!
bool compare(const Point& p1, const Point& p2, const Point& origin) {
    if(dist(p1, origin) == dist(p2, origin)) {
        return p1.x < p2.x || p1.y < p2.y;
    }
    return dist(p1, origin) < dist(p2, origin);
}
vector<Point> kClosest(vector<Point> &points, Point &origin, int k) {
    auto cmp = [&origin](const Point &p1, const Point &p2) -> bool {
        return compare(p1, p2, origin);
    };
    priority_queue<Point, vector<Point>, decltype(cmp)> maxq(cmp);
    for(auto p: points) {
        if(maxq.size() < k) {
            maxq.push(p);
        } else if(compare(p, maxq.top(), origin)) {  // 只判断距离不行，要有x,y判断
            maxq.pop();
            maxq.push(p);
        }
    }
    vector<Point> res;
    while (!maxq.empty()) {
        res.push_back(maxq.top());
        maxq.pop();
    }
    reverse(res.begin(), res.end());
    return res;
}
{% endhighlight %}
