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
