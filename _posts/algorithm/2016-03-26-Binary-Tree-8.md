---
layout: post
title:  "Binary Tree 8 - Others"
date:   2016-03-26 11:30:00
tags: [algorithm, leetcode, binary tree]
categories: Algorithm
---

### 1. [Balanced Binary Tree - Leetcode 110](https://leetcode.com/problems/balanced-binary-tree/)
```
Given a binary tree, determine if it is height-balanced.
For this problem, a height-balanced binary tree is defined as a binary tree in which the depth of the two subtrees of every node never differ by more than 1.
```

直观的方法,复杂度高,重复计算，某个节点会被遍历多次，求多次高度：
{% highlight C++ %}
int treeDepth(TreeNode *root) {
    if(root == NULL)
        return 0;
    int left = treeDepth(root->left);
    int right = treeDepth(root->right);
    return left>right? (left+1) : (right+1);    
}
bool isBalanced(TreeNode *root) {  
    if(root == NULL)
        return true;
    int leftD=0, rightD=0;
    leftD = treeDepth(root->left);
    rightD = treeDepth(root->right);
    // 算术运算符 > 关系运算符 > 逻辑运算符
    if( leftD-rightD > 1 || leftD-rightD < -1 )
        return false;
    return isBalanced(root->left) && isBalanced(root->right);
}
{% endhighlight %}
新方法：进一步简洁，设计核心函数比较重要
{% highlight C++ %}
bool isBalanced(TreeNode *root) {
    return balancedHeight (root) != -1;     //为了省去higt的参数
}
//返回值：-1代表不balanced，其他代表实际高度
int balancedHeight (TreeNode* root) {
    if (root == NULL) return 0;  // 终止条件 模板第1部分
    // 接下来处理left和right, 模板第2部分
    int left = balancedHeight (root->left);
    int right = balancedHeight (root->right);
    // 处理自己的 模板第3部分
    if (left == -1 || right == -1 || abs(left - right) > 1)
        return -1; // 剪枝
    return max(left, right) + 1; // 三方合并
}
{% endhighlight %}
