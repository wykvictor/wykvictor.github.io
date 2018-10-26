---
layout: post
title:  "Binary Tree 1 - DFS Traversal"
date:   2016-03-24 13:30:00
tags: [algorithm, leetcode, binary tree, traversal]
categories: Algorithm
---

> 二叉树问题: 考虑整棵树在该问题上的结果 和左右儿子在该问题上结果的联系

> DFS 总结：

![DFS-1](/res/DFS-1.png)

<table border="2" frame="box" cellspacing="0px" style="border-collapse:collapse" valign="center">
    <tr bgcolor="lightgreen">
        <th align="center">   Traverse   </th>
        <th align="center">Divide Conquer</th>
    </tr>
    <tr>
        <td align="center">Recursion Algorithm</td>
        <td align="center">Also Recursion</td>
    </tr>
    <tr>
        <td align="center">Result in parameter</td>
        <td align="center">Result in return value</td>
    </tr>
    <tr>
        <td align="center">Top down</td>
        <td align="center">Bottom up</td>
    </tr>
</table>

### 1. Binary Tree Preorder Traversal
Recurse(Traverse) - Template：
{% highlight C++ %}
vector<int> preorderTraversal(TreeNode *root) {
  vector<int> res;
  preorderTraversalCore(root, res);
  return res;
}
// 递归的三要素：定义，把root为跟的preorder加入res里面
void preorderTraversalCore(TreeNode *root, vector<int> &res) {
  // 三要素：递归结束
  if(root == NULL) // 比判断叶子节点要简洁些
      return;
  // 三要素：如何拆分问题（规模变小，问题相同）
  res.push_back(root->val);
  preorderTraversalCore(root->left, res); // res顺序添加，这2句不可并行
  preorderTraversalCore(root->right, res);
}
{% endhighlight %}
Recurse, version 2：Divide & Conquer
{% highlight C++ %}
vector<int> preorderTraversal(TreeNode *root) {
  vector<int> res;
  if(root == NULL)  return res;
  // Divide：好处，这2句可以并行
  vector<int> left = preorderTraversal(root->left);
  vector<int> right = preorderTraversal(root->right);
  // Conquer
  res.push_back(root->val);
  // 内存的copy，O(n)时间
  res.insert(res.end(), left.begin(), left.end());
  res.insert(res.end(), right.begin(), right.end());
  return res;
}
{% endhighlight %}
迭代_先序独特的方案，好：
{% highlight C++ %}
vector<int> preorderTraversal(TreeNode *root) {
  vector<int> res;
  if(root == NULL)    return res;
  stack<TreeNode *> s;
  s.push(root);
  while(!s.empty()) {
    TreeNode *node = s.top();
    res.push_back(node->val);
    s.pop();
    if(node->right)     // 先序的简单的方案：先压右，后压左
      s.push(node->right);
    if(node->left)
      s.push(node->left);
  }
  return res;
}
{% endhighlight %}
迭代2_几种遍历通用的方案，不大好理解：
{% highlight C++ %}
vector<int> preorderTraversal(TreeNode *root) {
  vector<int> res;
  stack<TreeNode *> s;
  TreeNode *node = root;
  while(!s.empty() || node) {  // s不空，或node有值
    if(node) {  // 只要node有，就访问，并转向左子树
      res.push_back(node->val);
      s.push(node);
      node = node->left;
    } else {    // 说明左边访问完了，转向右边
      node = s.top();  // stack弹出，不为访问，为跟踪到右边
      s.pop();
      node = node->right;
    }
  }
  return res;
}
{% endhighlight %}

### 2. Binary Tree Inorder Traversal
递归：
完全同上，只是把res.push_back(root->val);放到left和right中间

迭代_几种遍历通用的方案，最重要：
{% highlight C++ %}
vector<int> inorderTraversal(TreeNode *root) {
  vector<int> res;
  stack<TreeNode *> s;
  TreeNode *node = root;
  while(!s.empty() || node) {
    if(node) {  //如果node有
      s.push(node);
      node = node->left;      //一直到最左边
    } else {
      node = s.top();
      s.pop();
      res.push_back(node->val);   //在else里访问，就这一句位置不同
      node = node->right;
    }
  }
  return res;
}
{% endhighlight %}
另一种写法，更推荐：
{% highlight C++ %}
vector<int> inorderTraversal(TreeNode *root) {
  vector<int> res;
  if(root == NULL)  return res;
  stack<TreeNode *> s;
  TreeNode *node = root;
  while(node != NULL || !s.empty()) {
    // go to the leftest
    while(node != NULL) {
      s.push(node);
      node = node->left;
    }
    node = s.top();
    s.pop();
    res.push_back(node->val);
    node = node->right;
  }
  return res;
}
{% endhighlight %}

### 3. Binary Tree Postorder Traversal  - Hard
递归：
完全同上，只是把res.push_back(root->val);放到left和right最后

迭代_几种遍历通用的方案，更复杂一点，不很重要
{% highlight C++ %}
vector<int> postorderTraversal(TreeNode *root) {
    vector<int> res;
    stack<TreeNode *> s;
    TreeNode *preVisit=NULL, *cur=root;
    while(!s.empty() || cur) {
      //go to leftest
      while(cur != NULL) {
        s.push(cur);
        cur = cur->left;
      }
      cur = s.top(); // take the leftest out, but not pop yet
      // 右子树为空，或者访问过了
      if(cur->right == NULL || cur->right == preVisit) {
        res.push_back(cur->val);  // then take it
        preVisit = cur;
        s.pop();  // pop it
        cur = NULL;  // very Important!
      } else {
        cur = cur->right;
      }
    }
    return res;
  }
{% endhighlight %}
**Better solution**!!!

pre-order traversal is root-left-right, and post order is left-right-root.

Modify the code for pre-order to make it root-right-left, and then reverse the output so that we can get left-right-root：
{% highlight C++ %}
vector<int> postorderTraversal(TreeNode *root) {
  vector<int> res;
  if(root == NULL)    return res;
  stack<TreeNode *> s;
  s.push(root);
  while(!s.empty()) {
    TreeNode *node = s.top();
    res.push_back(node->val);
    s.pop();
    if(node->left)     //先序的简单的方案改编：先压左，后压右
      s.push(node->left);
    if(node->right)
      s.push(node->right);
  }
  reverse(res.begin(), res.end()); //反转 根右左 变成 左右根
  return res;
}
{% endhighlight %}
