---
layout: post
title:  "Binary Tree 6 - Tree Depth"
date:   2016-03-25 22:30:00
tags: [algorithm, leetcode, binary tree, depth]
categories: Algorithm
---

### 1. [Maximum depth of binary tree - Leetcode 104](https://leetcode.com/problems/maximum-depth-of-binary-tree/)
```
Given a binary tree, find its maximum depth.
The maximum depth is the number of nodes along the longest path from the root node down to the farthest leaf node.
```
{% highlight C++ %}
// 分治 比 遍历 这个题更简洁
int maxDepth(TreeNode *root) {
    if(root == NULL)
        return 0;
    return max(maxDepth(root->left), maxDepth(root->right))+1;
}
// 遍历，需要有个curdepth和最终res，和排列组合的搜索代码类似
void dfs(TreeNode * root, int &res, int curdepth) {
    if(root == nullptr) {
        return;
    }
    if(res < curdepth) res = curdepth;
    dfs(root->left, res, curdepth + 1);
    dfs(root->right, res, curdepth + 1);
}

int maxDepth(TreeNode * root) {
    // write your code here
    if(root == NULL) return 0;
    int res = 0;
    dfs(root, res, 1);
    return res;
}
{% endhighlight %}

### 2. [Minimum depth of binary tree - Leetcode 111](https://leetcode.com/problems/minimum-depth-of-binary-tree/)
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
  if(left == 0)   return right+1;
  if(right == 0)  return left+1;
  return min(left, right)+1;
}
{% endhighlight %}

### 3. [Balanced Binary Tree - Leetcode 110](https://leetcode.com/problems/balanced-binary-tree/)
```
Given a binary tree, determine if it is height-balanced.
For this problem, a height-balanced binary tree is defined as a binary tree in which the depth of the two subtrees of every node never differ by more than 1.
```

推荐(good coding style)：
{% highlight C++ %}
// we need this type has two fields:
struct Result {
  bool isbalanced;
  int height;
  Result(bool is, int h): isbalanced(is), height(h){}
};
Result isBalancedHelper(TreeNode *root) {
  if(root == NULL) {
      return Result(true, 0);
  }
  Result left = isBalancedHelper(root->left);
  Result right = isBalancedHelper(root->right);
  if(left.isbalanced == false || right.isbalanced == false)
      return Result(false, -1); // height is meaningless
  return Result(abs(left.height-right.height)<2,
                max(left.height,right.height)+1);
}

bool isBalanced(TreeNode *root) {
  return isBalancedHelper(root).isbalanced;
}
// 或者用c++ tuple
std::tuple<bool, int> core(TreeNode * root) {
    if(root == nullptr) return std::make_tuple(true, 0);
    auto left = core(root->left);
    auto right = core(root->right);
    if(!std::get<0>(left) || !std::get<0>(right)) return std::make_tuple(false, -1);
    int leftdepth = std::get<1>(left), rigthdepth = std::get<1>(right);
    return std::make_tuple(abs(leftdepth - rigthdepth) <= 1, max(leftdepth, rigthdepth) + 1);
}
bool isBalanced(TreeNode * root) {
    return std::get<0>(core(root));
}
{% endhighlight %}
如果不改返回的结构，那么可以遍历的过程中，记录下高度（思路参杂了便利/分治，容易写错）
{% highlight C++ %}
// 返回root的树，是否是balanced，顺便遍历出depth
bool core(TreeNode * root, int &depth) {
    if(root == nullptr) {
        depth = 0;
        return true;
    }
    int leftdepth, rightdepth;
    if(!core(root->left, leftdepth)) return false;
    if(!core(root->right, rightdepth)) return false;
    depth = max(leftdepth, rightdepth) + 1;  // 分治过程中，记录遍历depth的结果
    return (abs(leftdepth - rightdepth) <= 1);
}

bool isBalanced(TreeNode * root) {
    if(root == nullptr) return true;
    int depth;
    return core(root, depth);
}
{% endhighlight %}

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
//返回值：-1代表不balanced，其他代表实际高度(coding style不好)
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
