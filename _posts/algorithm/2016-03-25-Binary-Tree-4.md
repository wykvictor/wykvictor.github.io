---
layout: post
title:  "Binary Tree 4 - Compare Tree"
date:   2016-03-25 15:30:00
tags: [algorithm, leetcode, binary tree, Compare Tree]
categories: Algorithm
---

### 1. [Symmetric tree - Leetcode 101](https://leetcode.com/problems/symmetric-tree/)
```
Given a binary tree, check whether it is a mirror of itself (ie, symmetric around its center).
For example, this binary tree is symmetric:
    1
   / \
  2   2
 / \ / \
3  4 4  3
But the following is not:
    1
   / \
  2   2
   \   \
   3    3
```

递归版本：
{% highlight C++ %}
bool isSymmetric(TreeNode *root) {
    //递归版本
    if(root == NULL)    return true;
    return isSymmetricCore(root->left, root->right);
}
bool isSymmetricCore(TreeNode *leftNode, TreeNode *rightNode) {
    if(leftNode==NULL && rightNode==NULL) //先处理有个空的这种情况
        return true;
    if(leftNode==NULL || rightNode==NULL)
        return false;
    return (leftNode->val == rightNode->val) &&
            isSymmetricCore(leftNode->left, rightNode->right) &&
            isSymmetricCore(leftNode->right, rightNode->left);
}
{% endhighlight %}

迭代：
{% highlight C++ %}
bool isSymmetric(TreeNode *root) {
    //迭代版本，用队列吧，广度遍历
    if(root == NULL)
        return true;
    if(root->left==NULL && root->right==NULL)
        return true;
    if(root->left==NULL || root->right==NULL)
        return false;
    //下面开始正常处理流程
    queue<TreeNode *> q;
    q.push(root->left);
    q.push(root->right);
    while(!q.empty()) {
        TreeNode *leftNode = q.front();
        q.pop();
        TreeNode *rightNode = q.front();
        q.pop();
        if(leftNode->val != rightNode->val)
            return false;
        if(leftNode->left != NULL) {
            if(rightNode->right == NULL)
                return false;
            q.push(leftNode->left);
            q.push(rightNode->right);
        } else if(rightNode->right != NULL)
            return false;
        if(leftNode->right != NULL) {
            if(rightNode->left == NULL)
                return false;
            q.push(leftNode->right);
            q.push(rightNode->left);
        } else if(rightNode->left != NULL)
            return false;
    }
    return true;    //最终都判断完，才true
}
{% endhighlight %}
简化一下逻辑，空的也可以压入queue，就和递归思路一样了，里边就不需要判断4个点了，2个就够了：
{% highlight C++ %}
bool isSymmetric(TreeNode *root) {
    //迭代版本，用队列吧，广度遍历
    if(root == NULL)
        return true;
    //下面开始正常处理流程
    queue<TreeNode *> q;
    q.push(root->left); //NULL也压入
    q.push(root->right);
    while(!q.empty()) {
        TreeNode *leftNode = q.front();
        q.pop();
        TreeNode *rightNode = q.front();
        q.pop();
        if(leftNode==NULL && rightNode==NULL)
            continue;   //true时，继续判断
        if(leftNode==NULL || rightNode==NULL)   //之后，肯定都不空
            return false;
        if(leftNode->val != rightNode->val)
            return false;
         
        q.push(leftNode->left);
        q.push(rightNode->right);
        q.push(leftNode->right);
        q.push(rightNode->left);
    }
    return true;    //最终都判断完，才true
}
{% endhighlight %}

### 2. [Same tree - Leetcode 100](https://leetcode.com/problems/same-tree/)
```
Given two binary trees, write a function to check if they are equal or not.
Two binary trees are considered equal if they are structurally identical
and the nodes have the same value.
```

{% highlight C++ %}
bool isSameTree(TreeNode *p, TreeNode *q) {
    if(p == NULL && q == NULL)
        return true;
    if(p == NULL || q == NULL)
        return false;
    if(p->val != q->val)
        return false;
    return isSameTree(p->left, q->left) && isSameTree(p->right, q->right);
}
{% endhighlight %}
同样的，遍历方案也可：几乎和上边的一模一样
{% highlight C++ %}
bool isSameTree(TreeNode *p, TreeNode *q) {
    queue<TreeNode *> qnodes;
    qnodes.push(p);
    qnodes.push(q);
    while(!qnodes.empty()) {
        TreeNode *lnode = qnodes.front();
        qnodes.pop();
        TreeNode *rnode = qnodes.front();
        qnodes.pop();
        if(lnode==NULL && rnode==NULL)
            continue;
        if(lnode==NULL || rnode==NULL)
            return false;
        if(lnode->val != rnode->val)
            return false;
         
        qnodes.push(lnode->left);
        qnodes.push(rnode->left);
        qnodes.push(lnode->right);
        qnodes.push(rnode->right);
    }
    return true;
}
{% endhighlight %}

### 3. Subtree
```
You have two very large binary trees: T1, with millions of nodes, and T2 with hundreds of nodes.
Create an algorithm to decide if T2 is a subtree of T1. 
```
{% highlight C++ %}
bool subTree(TreeNode* t1, TreeNode* t2) { 
    if (!t2) return true; 
    if (!t1) return false; 
    if(t1->val == t2->val) 
        if(matchTree(t1,t2)) return true; 
    return subTree(t1->left,t2) || subtree(t1->right,t2); 
} 
bool matchTree(TreeNode* t1, TreeNode* t2) { //helper function
    if(!t1 && !t2) return true; 
    if(!t1 || !t2) return false; 
    if(t1->val != t2->val) return false; 
    return matchTree(t1->left,t2->left) && matchTree(t1->right,t2->right); 
}
{% endhighlight %}

### 4. [Tweaked Identical Binary Tree](http://www.lintcode.com/en/problem/tweaked-identical-binary-tree/#)
```
Check two given binary trees are identical or not. Assuming any number of tweaks are allowed. A tweak is defined as a swap of the children of one node in the tree.
```
{% highlight C++ %}
bool isTweakedIdentical(TreeNode* a, TreeNode* b) {
  if(a == NULL && b == NULL)
    return true;
  if(a == NULL || b == NULL)
    return false;
  if(a->val != b->val)
    return false;
  // 一致 或 对称
  return (isTweakedIdentical(a->left, b->left) && isTweakedIdentical(a->right, b->right))
    || (isTweakedIdentical(a->left, b->right) && isTweakedIdentical(a->right, b->left));
}
{% endhighlight %}
