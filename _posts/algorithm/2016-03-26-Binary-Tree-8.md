---
layout: post
title:  "Binary Tree 8 - BFS Traversal"
date:   2016-03-26 11:30:00
tags: [algorithm, leetcode, binary tree]
categories: Algorithm
---

### 1. [Binary Tree Level Order Traversal - Leetcode 102](https://leetcode.com/problems/binary-tree-level-order-traversal/)
```
Given a binary tree, return the level order traversal of its nodes' values. (ie, from left to right, level by level).
For example:
Given binary tree {3,9,20,#,#,15,7},
    3
   / \
  9  20
    /  \
   15   7
return its level order traversal as:
[
  [3],
  [9,20],
  [15,7]
]
```

方法1，直观简洁，每一层vector建立后，元素的存储不是按一行一行顺序来的，是先序的顺序，但最终结果对
{% highlight C++ %}
vector<vector<int> > levelOrder(TreeNode *root) {
    vector<vector<int> > res;   //因为要分层存储，所以遍历时加入level参数
    levelOrderCore(root, res, 0);
    return res;
}
void levelOrderCore(TreeNode *root, vector<vector<int> > &res, int level) {
    if(root == NULL)    return;
    if(res.size() <= level)
        res.push_back(vector<int>());   //这样子，空着初始化
 
    res[level].push_back(root->val);
    levelOrderCore(root->left, res, level+1);
    levelOrderCore(root->right, res, level+1);
}
{% endhighlight %}
方法2，迭代，也不错
{% highlight C++ %}
vector<vector<int> > levelOrder(TreeNode *root) {
    //迭代版本，BFS遍历变形!!（用2个队列，标记每一层!!）
    //时间复杂度 O(n)，空间复杂度 O(1)
    vector<vector<int> > res;
    if(root == NULL)  return res;
    queue<TreeNode *> cur,next; //分别记录本层和下一层的queue!!
    cur.push(root); //queue用push就好!
    while(!cur.empty()) {  //   //两层while，里边处理本行的
        vector<int> path;
        while(!cur.empty()) {    
            TreeNode *p = cur.front();  //队列
            cur.pop();
            path.push_back(p->val);
            if(p->left) next.push(p->left);
            if(p->right) next.push(p->right);
        }
        res.push_back(path);
        swap(next, cur);    //简洁!cur空了，next有值
    }
    return res;
}
{% endhighlight %}
方法3，迭代，O1空间，不需要2个queue，代码简洁：
{% highlight C++ %}
vector<vector<int> > levelOrder(TreeNode *root) {
    vector<vector<int> > res;
    if(root == NULL)
        return res;
    queue<TreeNode *> cur;    //只用1个就可以
    cur.push(root); //queue用push就好!
    while(!cur.empty()) {
        vector<int> path;
        for(int i=cur.size(); i>0; i--) {   //最开始取出queue里有多少个节点
            TreeNode *p = cur.front();  //队列
            cur.pop();
            path.push_back(p->val);
            if(p->left)     cur.push(p->left);
            if(p->right)    cur.push(p->right);
        }
        res.push_back(path);
    }
    return res;
}
{% endhighlight %}

### 2. [Binary Tree Level Order Traversal II - Leetcode 107](https://leetcode.com/problems/binary-tree-level-order-traversal-ii/)
```
Given a binary tree, return the bottom-up level order traversal of its nodes' values. (ie, from left to right, level by level from leaf to root).
For example:
Given binary tree {3,9,20,#,#,15,7},
    3
   / \
  9  20
    /  \
   15   7
return its bottom-up level order traversal as:
[
  [15,7],
  [9,20],
  [3]
]
```
{% highlight C++ %}
Same solution，只多下面一行：
reverse(res.begin(), res.end());
{% endhighlight %}

### 3. [Binary Tree Zigzag Level Order Traversal - Leetcode 103](https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/)
```
Given a binary tree, return the zigzag level order traversal of its nodes' values. (ie, from left to right, then right to left for the next level and alternate between).
For example:
Given binary tree {3,9,20,#,#,15,7},
    3
   / \
  9  20
    /  \
   15   7
return its zigzag level order traversal as:
[
  [3],
  [20,9],
  [15,7]
]
```

方法1的改编，奇数行前插
{% highlight C++ %}
vector<vector<int> > zigzagLevelOrder(TreeNode *root) {
  vector<vector<int> > res;
  zigzagLevelOrderCore(root, res, 0);
  return res;
}
void zigzagLevelOrderCore(TreeNode *root, vector<vector<int> > &res, int level) {
  if(root == NULL)    return;
  if(res.size() <= level)
    res.push_back(vector<int>());  //不够了，所以加
  //就这里加个判断即可，因为不知道某一行什么时候结束，所以不能最后用reverse，就插入的时候调整就行了
  if(level % 2 == 0)
    res[level].push_back(root->val);
  else
    res[level].insert(res[level].begin(), root->val);
  zigzagLevelOrderCore(root->left, res, level+1);
  zigzagLevelOrderCore(root->right, res, level+1);
}
{% endhighlight %}
方法3的改编：
{% highlight C++ %}
vector<vector<int>> zigzagLevelOrder(TreeNode *root) {
  vector<vector<int> > res;
  if(root == NULL)
    return res;
  queue<TreeNode *> cur;    //只用1个就可以
  cur.push(root); //queue用push就好!
  bool reversed = false;
  while(!cur.empty()) {
      vector<int> path;
      for(int i=cur.size(); i>0; i--) {   //最开始取出queue里有多少个节点
        TreeNode *p = cur.front();  //队列
        cur.pop();
        path.push_back(p->val);
        if(p->left)     cur.push(p->left);
        if(p->right)    cur.push(p->right);
      }
      if(reversed) {
        reverse(path.begin(), path.end());
      }
      reversed = !reversed;
      res.push_back(path);
  }
  return res;
}
{% endhighlight %}
用stack，不需要调用reverse了，优化时间,好！
{% highlight C++ %}
vector<vector<int>> zigzagLevelOrder(TreeNode *root) {
  vector<vector<int> > res;
  if(root == NULL)
    return res;
  stack<TreeNode *> cur;
  stack<TreeNode *> next;
  cur.push(root);
  bool reversed = true;
  while(!cur.empty()) {
    vector<int> path;
    while(!cur.empty()) {
      TreeNode *p = cur.top();
      cur.pop();
      path.push_back(p->val);
      if(reversed) {
        if(p->left)  next.push(p->left);
        if(p->right)  next.push(p->right);
      } else {
        if(p->right)  next.push(p->right);
        if(p->left)  next.push(p->left);
      }
    }
    reversed = !reversed;
    res.push_back(path);
    swap(cur, next);
  }
  return res;
}
{% endhighlight %}
