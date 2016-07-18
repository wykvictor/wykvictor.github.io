---
layout: post
title:  "Linked List 1 - "
date:   2016-04-22 18:30:00
tags: [algorithm, leetcode, dynamic programming, dp]
categories: Algorithm
---

### 1. [Reverse Linked List](http://www.lintcode.com/en/problem/reverse-linked-list/)

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

### 2. [Reverse Linked List II](http://www.lintcode.com/en/problem/reverse-linked-list-ii/)
```
Reverse a linked list from position m to n.
1 ≤ m ≤ n ≤ length of list.
```
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

