---
layout: post
title:  "Linked List 1 - Basic Modules"
date:   2016-04-22 18:30:00
tags: [algorithm, leetcode, linked list]
categories: Algorithm
---

> Use modules packaged in different functions

* Insert a Node in Sorted List
* Remove a Node from Linked List
* Reverse a Linked List
* Merge Two Linked Lists
* Middle of a Linked List

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

### 3. [Reorder List](http://www.lintcode.com/en/problem/reorder-list/)
```
Given 1->2->3->4->null, reorder it to 1->4->2->3->null.
```
{% highlight C++ %}
ListNode *getMid(ListNode *head) {
  if (head == NULL) {
    return NULL;
  }
  ListNode *slow = head, *fast = head->next;    // !
  while (fast != NULL && fast->next != NULL) {  // !
    slow = slow->next;
    fast = fast->next->next;
  }
  return slow;  // ! 返回上一个!!!
}
ListNode *reverse(ListNode *head) {
  ListNode *prev = NULL, *cur = head;
  while (cur != NULL) {
    ListNode *temp = cur->next;
    cur->next = prev;
    prev = cur;
    cur = temp;
  }
  return prev;
}
void reorderList(ListNode *head) {
  if (head == NULL) return;
  // module 1: get mid
  ListNode *mid = getMid(head);
  // module 2: reverse 2
  ListNode *reversed = reverse(mid->next);
  mid->next = NULL;  // 封口!
  // module 3: merge 2 list
  ListNode *p1 = head, *p2 = reversed;
  while (p1 != NULL && p2 != NULL) {
    ListNode *t1 = p1->next;
    p1->next = p2;
    p1 = t1;
    ListNode *t2 = p2->next;
    p2->next = p1;
    p2 = t2;
  } // l1比l2长
  return;
}
{% endhighlight %}

### 4. [Sort List](http://www.lintcode.com/en/problem/sort-list/)
Merge Sort: 时间O(nlogn)，局部有序->整体有序，稳定排序
{% highlight C++ %}
ListNode *sortList(ListNode *head) {
  // 退出条件
  if (head == NULL || head->next == NULL) {
    return head;
  }
  ListNode *mid = getMid(head);  //module 1

  ListNode *right = sortList(mid->next);
  mid->next = NULL;  // 封口,所以right放在left前
  ListNode *left = sortList(head);

  return mergeTwoLists(left, right); //module 2
}
{% endhighlight %}
Quick Sort: 时间O(nlogn)，最坏O(n^2)，整体有序->局部有序，不稳定排序
{% highlight C++ %}
int partition(vector<int> &A, int low, int high) {
    int pivot = A[low];
    while(low < high) {
        if(low < high && pivot <= A[high]) high--;
        A[low] = A[high];  // A[low] 可以用，接下来A[high]也可以被占
        if(low < high && pivot >= A[low]) low++;
        A[high] = A[low]; 
    }
    A[low] = pivot;
    return low;
}
void quickSort(vector<int> &A, int low, int high) {
    if(low >= high) return;  // 注意这个条件！！
    int mid = partition(A, low, high);
    quickSort(A, low, mid-1);
    quickSort(A, mid+1, high);
}
{% endhighlight %}

{% highlight C++ %}
ListNode *getTail(ListNode *head) {
    if(head == NULL)  return NULL;
    ListNode *cur = head;
    // ！判断条件
    while(cur->next != NULL) {
        cur = cur->next;
    }
    return cur;
}
// !!极端情况: 如果3个list有NULL
ListNode *concat(ListNode *l1, ListNode *l2, ListNode *l3) {
  ListNode *head = l1;

  ListNode *tail1 = getTail(l1);
  if (tail1 == NULL) {
    head = l2;
  } else {
    tail1->next = l2;
  }
  ListNode *tail2 = getTail(l2);
  if (tail2 == NULL) {
    tail1->next = l3;
  } else {
    tail2->next = l3;
  }

  return head;
}

ListNode *sortList(ListNode *head) {
  // 退出条件
  if (head == NULL || head->next == NULL) {
    return head;
  }

  // Partition:
  ListNode *mid = getMid(head);  // O(n)
  // Array快排区别：没有i,j左右游标，得有mid的链表，才能左右分
  ListNode dummy_min(0), dummy_max(0), dummy_mid(0);
  ListNode *p = head, *pmin = &dummy_min, *pmax = &dummy_max,
           *pmid = &dummy_mid;
  while (p != NULL) {
    if (p->val < mid->val) {
      pmin->next = p;
      pmin = pmin->next;
    } else if (p->val > mid->val) {
      pmax->next = p;
      pmax = pmax->next;
    } else {
      pmid->next = p;
      pmid = pmid->next;
    }
    p = p->next;
  }
  pmin->next = NULL;
  pmax->next = NULL;
  pmid->next = NULL;

  // 递归
  ListNode *left = sortList(dummy_min.next);
  ListNode *right = sortList(dummy_max.next);

  // 3者连接
  return concat(left, dummy_mid.next, right);
}

// 简洁方式
void printlist(ListNode *head) {
    std::cout << "printlist:" << std::endl;
    while(head != NULL) {
        std::cout << head->val << "->";
        head = head->next;
    }
    std::cout << "end" << std::endl;
}

ListNode * sortList(ListNode * head) {
    // write your code here
    if(head == NULL || head->next == NULL)  return head;
    ListNode smallhead(-1), bighead(-1);
    ListNode *small = &smallhead, *big = &bighead;
    ListNode *cur = head->next;
    ListNode *pivot = head;  // 头作为pivot
    while(cur != NULL) {
        if(cur->val < pivot->val) {
            small->next = cur;
            small = cur;
        } else {
            big->next = cur;
            big = cur;
        }
        cur = cur->next;
    }
    small->next = NULL;
    big->next = NULL;
    small = sortList(smallhead.next);
    big = sortList(bighead.next);

    // concat small, pivot, big
    pivot->next = big;
    ListNode *tailsmall = getTail(small);
    if(tailsmall != NULL) {
        tailsmall->next = pivot;
        return small;  // ！！！不是smallhead.next
    } else {
        return pivot;
    }
}
{% endhighlight %}

### 5. [Reverse Nodes in k-Group](http://www.lintcode.com/en/problem/reverse-nodes-in-k-group/)
```
Given this linked list: 1->2->3->4->5
For k = 2, you should return: 2->1->4->3->5
For k = 3, you should return: 3->2->1->4->5
```
{% highlight C++ %}
ListNode *reverseKGroup(ListNode *head, int k) {
  if (k <= 1) return head;
  int len = getLength(head);
  // if (k >= len) return reverse(head); //下面包含了此情况

  ListNode dummy(0), *p = &dummy;
  dummy.next = head;
  // 根据长度，算出区间数，保证不越界，简化逻辑
  for (int i = 0; i < len / k; i++) {
    ListNode *tail = p->next;  // 记录尾部
    // 翻转k个节点
    ListNode *prev = NULL, *cur = p->next;
    for (int i = 0; i < k; i++) {
      ListNode *temp = cur->next;
      cur->next = prev;
      // 两个好基友,一起往后走
      prev = cur;
      cur = temp;
    }
    p->next = prev;  // cur成了尾巴的后一个
    tail->next = cur;
    p = tail;
  }

  return dummy.next;
}
{% endhighlight %}