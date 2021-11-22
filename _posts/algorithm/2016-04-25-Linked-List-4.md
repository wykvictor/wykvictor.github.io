---
layout: post
title:  "Linked List 4 - Others"
date:   2016-04-25 18:30:00
tags: [algorithm, leetcode, linked list]
categories: Algorithm
---

### 1. [Important! Merge k Sorted Lists](http://www.lintcode.com/en/problem/merge-k-sorted-lists/)
```
Merge k sorted linked lists and return it as one sorted list.

Analyze and describe its complexity. N is the totle numbers.
```

Merge lists two by two I: 顺序merge, O(Nk)，平均每个点都得合并K次
{% highlight C++ %}
ListNode *mergeKLists(vector<ListNode *> &lists) {
  if (lists.size() == 0) return NULL;
  ListNode *head = lists[0];  // !!initialize
  for (int i = 1; i < lists.size(); i++) {
    head = mergeTwoLists(head, lists[i]);
  }
  return head;
}
{% endhighlight %}
Merge lists two by two II: 两两merge, O(Nlogk)，每个点合并logk次
{% highlight C++ %}
ListNode *mergeKLists(vector<ListNode *> &lists) {
  if (lists.size() == 0) return NULL;
  vector<ListNode *> cur(lists);

  while (cur.size() > 1) {
    vector<ListNode *> tmp;
    for (int i = 0; i < cur.size(); i += 2) {
      if (i == cur.size() - 1) {
        tmp.push_back(cur[i]);
      } else {
        tmp.push_back(mergeTwoLists(cur[i], cur[i + 1]));
      }
    }
    cur = tmp;
  }

  return cur[0];
}
{% endhighlight %}
Divide Conquer: 上一解法的递归版，O(Nlogk)，每个点合并logk次
{% highlight C++ %}
ListNode *mergeKListsHelper(vector<ListNode *> &lists, int left, int right) {
  if (right < left) return NULL;
  if (right == left) return lists[left];
  // divide
  int mid = left + (right - left) / 2;
  ListNode *leftList = mergeKListsHelper(lists, left, mid);
  ListNode *rightList = mergeKListsHelper(lists, mid + 1, right);
  // conquer
  return mergeTwoLists(leftList, rightList);
}

ListNode *mergeKLists(vector<ListNode *> &lists) {
  if (lists.size() == 0) return NULL;
  return mergeKListsHelper(lists, 0, lists.size() - 1);
}
{% endhighlight %}
Priority Queue(Heap): 推荐做法，K个元素的最小堆，平均每个点进出堆1次, O(Nlogk)
{% highlight C++ %}
struct cmp {
  bool operator()(ListNode *a, ListNode *b) {
    return a->val > b->val;  // a b是前后两个元素，true则其需交换位置
  }
};

ListNode *mergeKLists(vector<ListNode *> &lists) {
  if (lists.size() == 0) return NULL;
  priority_queue<ListNode *, vector<ListNode *>, cmp> q;  // !
  // 建堆
  for (int i = 0; i < lists.size(); i++) {
    if (lists[i] != NULL) q.push(lists[i]);
  }
  // 依次取元素
  ListNode dummy(0), *node = &dummy;
  while (!q.empty()) {
    ListNode *tmp = q.top();
    q.pop();
    node->next = tmp;
    node = node->next;
    if (tmp->next) q.push(tmp->next);
  }
  return dummy.next;
}
// 另：lamda函数初始化queue
ListNode *mergeKLists(vector<ListNode *> &lists) {
  // write your code here
  if(lists.empty()) return NULL;
  if(lists.size() == 1) return lists[0];
  // 可调用对象cmp
  auto cmp = [](ListNode *l1, ListNode *l2) -> bool {
      return l1->val > l2->val;  // 升序，就是>
  };
  // lambda表达式的匿名类型是没有默认构造函数的，所以需要传进去
  priority_queue<ListNode*, vector<ListNode*>, decltype(cmp)> q(cmp);
  for(auto l: lists) {
      if(l != NULL) q.push(l);
  }
  ListNode head(-1);
  ListNode *cur = &head;
  while(!q.empty()) {
      ListNode *topnode = q.top();
      q.pop();
      cur->next = topnode;
      cur = cur->next;
      if(topnode->next) q.push(topnode->next);
  }
  cur->next = NULL;
  return head.next;
}
{% endhighlight %}

### 2. [Copy List with Random Pointer](http://www.lintcode.com/en/problem/copy-list-with-random-pointer/)
{% highlight C++ %}
RandomListNode *copyRandomList(RandomListNode *head) {
  if (head == NULL) return NULL;

  // 1. copy each node behide
  RandomListNode *p = head;
  while (p != NULL) {
    // !!!临时变量不可以，RandomListNode copyed(p->label);
    RandomListNode *copyed = new RandomListNode(p->label);  // new在堆上，不会释放
    RandomListNode *after = p->next;
    p->next = copyed;
    p->next->next = after;
    p = after;
  }

  // 2. random refer to right place
  p = head;
  while (p != NULL) {
    if (p->random != NULL) {  // !!can point to NULL
      p->next->random = p->random->next;
    }
    p = p->next->next;
  }

  // 3. split into 2 lists
  p = head;
  RandomListNode *copyHead = head->next;
  RandomListNode *q = copyHead;
  while (q->next != NULL) {
    p->next = q->next;
    p = p->next;
    q->next = p->next;
    q = q->next;
  }
  // 封口
  p->next = NULL;
  return copyHead;
}
{% endhighlight %}
传统的hash map方法:
{% highlight C++ %}
RandomListNode* copyRandomList(RandomListNode* head) {
  // write your code here
  unordered_map<RandomListNode*, RandomListNode*> old2new;
  RandomListNode* dummy = new RandomListNode(-1);
  RandomListNode* tmp = head;
  RandomListNode* curr = dummy;
  while (tmp) {
    RandomListNode* newNode = new RandomListNode(tmp->label);
    old2new[tmp] = newNode;
    curr->next = newNode;
    curr = curr->next;
    tmp = tmp->next;
  }
  tmp = head;
  while (tmp) {
    if (tmp->random) {
      old2new[tmp]->random = old2new[tmp->random];
    }
    tmp = tmp->next;
  }
  return dummy->next;
}
{% endhighlight %}
