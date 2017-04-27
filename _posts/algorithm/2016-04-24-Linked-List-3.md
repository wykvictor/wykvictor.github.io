---
layout: post
title:  "Linked List 3 - Fast Slow Pointers"
date:   2016-04-24 18:30:00
tags: [algorithm, leetcode, linked list, 2 pointers]
categories: Algorithm
---

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
int getLength(ListNode *head) {
  int length = 0;
  ListNode *p = head;
  while (p != NULL) {
    length++;
    p = p->next;
  }
  return length;
}

ListNode *rotateRight(ListNode *head, int k) {
  if (head == NULL) return NULL;

  int len = getLength(head);
  k = k % len;
  if (k == 0) return head;  // 加上这句更保险！

  ListNode *fast = head, *slow = head;
  for (int i = 0; i < k; i++) {
    fast = fast->next;
  }
  while (fast->next != NULL) {
    fast = fast->next;
    slow = slow->next;
  }
  fast->next = head;
  ListNode *newHead = slow->next;
  slow->next = NULL;
  return newHead;
}
{% endhighlight %}

### 3. [Insert into a Cyclic Sorted List](http://www.lintcode.com/en/problem/insert-into-a-cyclic-sorted-list/)
```
3->5->1 is a cyclic list, so 3 is next node of 1.
3->5->1 is same with 5->1->3
insert a value 4: Return 5->1->3->4
```
{% highlight C++ %}
ListNode* insert(ListNode* node, int x) {
  ListNode* X = new ListNode(x);
  if (node == NULL) {  //特殊情况
    X->next = X;       //注意，收尾
    return X;
  }
  ListNode* pre = node, * cur = node->next;
  while (cur != node) {                      // 终止条件!
    if (pre->val > cur->val) {               // 首尾相接处
      if (x >= pre->val || x <= cur->val) {  // 注意有等于
        break;
      }
    } else {
      if (x >= pre->val && x <= cur->val) {  // 注意有等于!
        break;
      }
    }
    pre = cur;
    cur = cur->next;
  }
  pre->next = X;
  X->next = cur;
  return node;
}
{% endhighlight %}
