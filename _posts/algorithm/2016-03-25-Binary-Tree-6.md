---
layout: post
title:  "Binary Tree 6 - Tree Depth"
date:   2016-03-25 22:30:00
tags: [algorithm, leetcode, binary tree, depth]
categories: Algorithm
---

### 1. [Minimum depth of binary tree - Leetcode 111](https://leetcode.com/problems/minimum-depth-of-binary-tree/)
```
Given a binary tree, find its minimum depth.
The minimum depth is the number of nodes along the shortest path from the root node down to the nearest leaf node.
```
{% highlight C++ %}
int minDepth(TreeNode *root) {
    if(root == NULL)
        return 0;
    int left = minDepth(root->left);
    int right = minDepth(root->right);
    // 只要没有2个孩子，就是另一个的+1
    return (left==0 || right==0) ? max(left, right)+1 : min(left, right)+1;
}
{% endhighlight %}

### 2. [Maximum depth of binary tree - Leetcode 104](https://leetcode.com/problems/maximum-depth-of-binary-tree/)
```
Given a binary tree, find its maximum depth.
The maximum depth is the number of nodes along the longest path from the root node down to the farthest leaf node.
```
{% highlight C++ %}
int maxDepth(TreeNode *root) {
    if(root == NULL)
        return 0;
    return max(maxDepth(root->left), maxDepth(root->right))+1;
}
{% endhighlight %}
