---
layout: post
title:  "Linked List 4 - Fast Slow Pointers"
date:   2016-04-24 18:30:00
tags: [algorithm, leetcode, linked list, 2 pointers]
categories: Algorithm
---

* ListNode *node; 只存储指向的地址，只有修改next才有意义
* Dummy Node: 当链表结构变化，头结点可能改变时

### 1. [Linked List Cycle](http://www.lintcode.com/en/problem/linked-list-cycle/)
{% highlight C++ %}
bool hasCycle(ListNode *head) {
  if (head == NULL) return false;
  ListNode *slow = head, *fast = head->next;
  while (fast != NULL && fast->next != NULL) {
    if (fast == slow) {
      return true;
    }
    slow = slow->next;
    fast = fast->next->next;
  }
  return false;
}
{% endhighlight %}
求交点：
{% highlight C++ %}
ListNode *detectCycle(ListNode *head) {
  if (head == NULL || head->next == NULL) return NULL;
  ListNode *slow = head, *fast = head->next;
  while (fast != NULL && fast->next != NULL) {
    if (fast == slow) {
      break;
    }
    slow = slow->next;
    fast = fast->next->next;
  }
  if (fast == NULL || fast->next == NULL) return NULL;
  slow = head;
  fast = fast->next;
  while (fast != slow) {
    fast = fast->next;
    slow = slow->next;
  }
  return fast;
}
{% endhighlight %}

### 2. [Rotate List](http://www.lintcode.com/en/problem/rotate-list/)
```
Given a list, rotate the list to the right by k places, where k is non-negative.
Given 1->2->3->4->5 and k = 2, return 4->5->1->2->3.
```
{% highlight C++ %}

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
