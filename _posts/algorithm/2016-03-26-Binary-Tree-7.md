---
layout: post
title:  "Binary Tree 7 - Reshaping Tree"
date:   2016-03-26 10:30:00
tags: [algorithm, leetcode, binary tree, reshape]
categories: Algorithm
---

### 1. [Flatten Binary Tree to Linked List - Leetcode 114](https://leetcode.com/problems/flatten-binary-tree-to-linked-list/)
```
Given a binary tree, flatten it to a linked list in-place.
For example,
Given
         1
        / \
       2   5
      / \   \
     3   4   6
The flattened tree should look like:
   1
    \
     2
      \
       3
        \
         4
          \
           5
            \
             6
```

递归版1，变左变右，左插右前边 时间复杂度 O(n)，空间复杂度 O(logn)
{% highlight C++ %}
void flatten(TreeNode *root) {
    //弄成单向链表即可，类似preorder的遍历，递归
    //root的左右先做这个动作，然后把left插入到root和right之间即可
    if(root == NULL)    return;
    flatten(root->left);
    flatten(root->right);
    if(root->left == NULL)  return; //若left为空，就已经成了，不需要下边的插的步骤
    TreeNode *p = root->left;
    while(p->right != NULL)  //一直找到left的最后一个节点 
        p = p->right;
    p->right = root->right;
    root->right = root->left;
    root->left = NULL;    //必须封口，不能乱来
}
{% endhighlight %}
递归版2，超简洁递归法!思路其实也是左插右前边
{% highlight C++ %}
void flatten(TreeNode *root) {
    flattenCore(root, NULL);
}
//把root所代表的树变成链表后，tail跟在该链表后面
TreeNode *flattenCore(TreeNode *root, TreeNode *tail) {
    if(root == NULL) return tail; //若root没有，返回的就是root后边的那个tail
    TreeNode *rightRoot = flattenCore(root->right, tail);   //整体root的tail就是right的tail
    TreeNode *leftRoot = flattenCore(root->left, rightRoot); //右边的root就是左边的tail，要连起来
    root->right = leftRoot;
    root->left = NULL;  //封口
    return root;    //返回的是root!
}
{% endhighlight %}
迭代版，感觉更好理解，推荐方法! 时间复杂度 O(n)，空间复杂度 O(logn)
{% highlight C++ %}
void flatten(TreeNode *root) {
    //迭代，用栈!! 前序遍历，非常方便的方法，应该效率较高也
    if(root == NULL)    return;
    stack<TreeNode *> s;
    s.push(root);
    while(!s.empty()) {
        TreeNode *temp = s.top();
        s.pop();
        if(temp->right)  s.push(temp->right);
        if(temp->left)   s.push(temp->left);
        if(!s.empty())
            temp->right = s.top(); //串接到下一个要访问的元素
        temp->left = NULL;  //仍然要封口!
    }
}
{% endhighlight %}

### 2. [Populating Next Right Pointers in Each Node - Leetcode 116](https://leetcode.com/problems/populating-next-right-pointers-in-each-node/)
```
Given a binary tree:
struct TreeLinkNode {
  TreeLinkNode *left;
  TreeLinkNode *right;
  TreeLinkNode *next;
}
Populate each next pointer to point to its next right node. If there is no next right node, the next pointer should be set to NULL.
Initially, all next pointers are set to NULL.

Note:
You may only use constant extra space.
You may assume that it is a perfect binary tree (ie, all leaves are at the same level, and every parent has two children).
```

自己写的迭代算法！
{% highlight C++ %}
void connect(TreeLinkNode *root) {
    //O(1) 空间，从上一层得到下一层
    if(root == NULL)    return;
    TreeLinkNode *node=root;
    while(node->left != NULL) { //处理到叶子节点就返回
        //伪节点，是为了实现link->next = p->left; 要不还得特殊判断这一行
        TreeLinkNode *link = new TreeLinkNode(-1);  // 有memory问题，需要delete
        for(TreeLinkNode *p=node; p!=NULL; p=p->next) {
            link->next = p->left;
            link->next->next = p->right;
            link = link->next->next;
        }
        node = node->left;  //下一行开始
    }
}
{% endhighlight %}
DFS递归方法，思路简洁!
{% highlight C++ %}
void connect(TreeLinkNode *root) {
    if(root == NULL)
        return;
    connectCore(root->left, root->right);
}
void connectCore(TreeLinkNode *leftSibling, TreeLinkNode *rightSibling) {
    if(leftSibling == NULL) //相当于判断rightSibling了也
        return;
    leftSibling->next = rightSibling;
    if(leftSibling->left == NULL)   //同样判断一个即可
        return;
    connectCore(leftSibling->left, leftSibling->right);
    connectCore(rightSibling->left, rightSibling->right);
    //两者之间
    connectCore(leftSibling->right, rightSibling->left);
}
{% endhighlight %}

### 3. [Populating Next Right Pointers in Each Node II - Leetcode 117](https://leetcode.com/problems/populating-next-right-pointers-in-each-node-ii/)
```
Follow up for problem "Populating Next Right Pointers in Each Node".
What if the given tree could be any binary tree? Would your previous solution still work?
```

自己写的迭代算法，仍然可以用

区别：需要记录下linkhead，因为不一定head->left有值；内部判断p->left
{% highlight C++ %}
void connect(TreeLinkNode *root) {
    //O(1) 空间，从上一层得到下一层
    if(root == NULL)    return;
    TreeLinkNode *head = root;  //伪节点，每一层开头
    while(head != NULL) { //处理到叶子节点就返回
        //下一层开头伪节点，是为了串起下边一排
        TreeLinkNode linkhead(-1);
        TreeLinkNode *link = linkhead;
        for(TreeLinkNode *p=head; p!=NULL; p=p->next) {
            if(p->left) {
                link->next = p->left;
                link = link->next;
            }
            if(p->right) {
                link->next = p->right;
                link = link->next;
            }
        }
        head = linkhead->next;  //head下一行开始
    }
}
{% endhighlight %}
递归方法，其实思路和上边一样：
{% highlight C++ %}
void connect(TreeLinkNode *root) {
    if(root == NULL)    return;
    TreeLinkNode dummy(-1); //每一层的，伪头结点
    for(TreeLinkNode *curr=root, *prev=&dummy; curr; curr=curr->next) { //每一层，从头到尾进行链接
        if(curr->left) {
            prev->next = curr->left;
            prev = prev->next;
        }
        if(curr->right) {
            prev->next = curr->right;
            prev = prev->next;
        }
    }
    connect(dummy.next);    //每一层的开头的节点
}
{% endhighlight %}
