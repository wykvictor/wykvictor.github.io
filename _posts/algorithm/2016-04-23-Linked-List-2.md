---
layout: post
title:  "Linked List 2 - Dummy Node"
date:   2016-04-23 18:30:00
tags: [algorithm, leetcode, linked list, dummy node]
categories: Algorithm
---

* ListNode *node; 只存储指向的地址，只有修改next才有意义
* Dummy Node: 当链表结构变化，头结点可能改变时

### 1. [Reverse Linked List II](http://www.lintcode.com/en/problem/reverse-linked-list-ii/)
```
Reverse a linked list from position m to n.
1 ≤ m ≤ n ≤ length of list.
```

引子：[Reverse Linked List](http://www.lintcode.com/en/problem/reverse-linked-list/)
{% highlight C++ %}
ListNode *reverse(ListNode *head) {
  ListNode *prev = NULL, *cur = head;
  while (cur != NULL) {
    ListNode *temp = cur->next;
    cur->next = prev;
    // 两个好基友,一起往后走
    prev = cur;
    cur = temp;
  }
  return prev;
}
{% endhighlight %}
本题：
{% highlight C++ %}
// k is not the index, but kth
ListNode *getKth(ListNode *head, int k) {
  ListNode *node = head;
  for (int i = 1; i < k; i++) {
    if (node == NULL) {
      return NULL;
    }
    node = node->next;
  }
  return node;
}

ListNode *reverseBetween(ListNode *head, int m, int n) {
  ListNode dummy(0);
  dummy.next = head;

  // 1. find m
  ListNode *prevM = getKth(&dummy, m);
  ListNode *prev = prevM->next, *cur = prevM->next->next;

  // 2. reverse m<->n
  for (int i = m; i < n; i++) {
    ListNode *temp = cur->next;
    cur->next = prev;
    // 两个好基友,一起往后走
    prev = cur;
    cur = temp;
  }

  // 3. link the list: m,n接前后
  prevM->next->next = cur;  // 也可以事先记录下prevM->next
  prevM->next = prev;

  return dummy.next;
}
{% endhighlight %}

### 2. [Remove Duplicates from Sorted List II](http://www.lintcode.com/en/problem/reverse-linked-list-ii/)
```
Given a sorted linked list, delete all nodes that have duplicate numbers.
Given 1->1->1->2->3, return 2->3.
```
{% highlight C++ %}
ListNode *deleteDuplicates(ListNode *head) {
  ListNode dummy(0);
  dummy.next = head;  // 大部分链表都需要这3行

  ListNode *prev = &dummy, *cur = head;  // need this 2
  // 取next的时候，一定判断cur != NULL !!!
  while (cur != NULL && cur->next != NULL) {  // 根据下边的if决定
    if (cur->val == cur->next->val) {
      // delete, need while,删所有的
      int val = cur->val;
      while (cur != NULL && cur->val == val) {
        ListNode *d = cur;
        cur = cur->next;
        delete d;
      }
      prev->next = cur;
    } else {
      prev = cur;
      cur = cur->next;
    }
  }
     
  return dummy.next;
}
{% endhighlight %}

### 3. [Partition List](http://www.lintcode.com/en/problem/partition-list/)
```
Given a linked list and a value x, partition it such that all nodes less than x come before nodes greater than or equal to x.

You should preserve the original relative order of the nodes in each of the two partitions.
Given 1->4->3->2->5->2->null and x = 3,
return 1->2->2->4->3->5->null.
```
{% highlight C++ %}
ListNode *partition(ListNode *head, int x) {
  ListNode dummy_min(0);
  ListNode dummy_max(0);
  ListNode *p = head, *pmin = &dummy_min, *pmax = &dummy_max;
  while (p != NULL) {
    if (p->val < x) {
      pmin->next = p;
      pmin = pmin->next;
    } else {
      pmax->next = p;
      pmax = pmax->next;
    }
    p = p->next;
  }
  pmin->next = dummy_max.next;
  pmax->next = NULL;

  return dummy_min.next;
}
{% endhighlight %}

### 4. [Merge Two Sorted Lists](http://www.lintcode.com/en/problem/merge-two-sorted-lists/)
{% highlight C++ %}
ListNode *mergeTwoLists(ListNode *l1, ListNode *l2) {
  ListNode dummy(0);
  ListNode *p = &dummy;

  while (l1 != NULL && l2 != NULL) {
    if (l1->val < l2->val) {
      p->next = l1;
      l1 = l1->next;
    } else {
      p->next = l2;
      l2 = l2->next;
    }
    p = p->next;
  }
  if (l1 != NULL) {
    p->next = l1;
  }
  if (l2 != NULL) {
    p->next = l2;
  }

  return dummy.next;
}
{% endhighlight %}
