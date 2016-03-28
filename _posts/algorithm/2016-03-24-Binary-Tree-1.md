---
layout: post
title:  "Binary Tree 1 - Traversal"
date:   2016-03-24 13:30:00
tags: [algorithm, leetcode, binary tree, traversal]
categories: Algorithm
---

### 1. Binary Tree Preorder Traversal
递归：
{% highlight C++ %}
vector<int> preorderTraversal(TreeNode *root) {
    vector<int> res;
    preorderTraversalCore(root, res);
    return res;
}
void preorderTraversalCore(TreeNode *root, vector<int> &res) {
    if(root == NULL)
        return;
    res.push_back(root->val);
    preorderTraversalCore(root->left, res);
    preorderTraversalCore(root->right, res);
}
{% endhighlight %}
迭代_先序独特的方案：
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
{% highlight C++ %}
完全同上，只是把res.push_back(root->val);放到left和right中间
{% endhighlight %}
迭代_几种遍历通用的方案：
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

### 3. Binary Tree Postorder Traversal  - Hard
递归：
{% highlight C++ %}
完全同上，只是把res.push_back(root->val);放到left和right最后
{% endhighlight %}
迭代_几种遍历通用的方案，更复杂一点
{% highlight C++ %}
vector<int> postorderTraversal(TreeNode *root) {
    vector<int> res;
    stack<TreeNode *> s;
    TreeNode *prevVisit=NULL, *cur=root;
    while(!s.empty() || cur) {
        for(; cur->left; cur = cur->left)   //一直找到最左边
            s.push(cur);
        //当前节点为空，且右子树不用访问了==>那么访问该节点
        while(cur && (cur->right == NULL || cur->right == prevVisit)) {
            res.push_back(cur->val);
            prevVisit = cur; //记录一下
            if(s.empty())   //结束!!，总体来说，有2个叶节点的父节点都被压入了2次
                return res;
            cur = s.top();  //搞个新的出来,之前需要判断下
            s.pop();
        }
        s.push(cur);    //则依次访问右边的
        cur = cur->right;
    }
    return res;
}
{% endhighlight %}
改变了输入树的结构，做标记。但是代码思路简洁：
{% highlight C++ %}
vector<int> postorderTraversal(TreeNode *root) {
    vector<int> res;
    stack<TreeNode *> s;
    if(root == NULL)    return res;
    s.push(root);
    while(!s.empty()) {
        TreeNode *cur = s.top();
        if(cur->left==NULL && cur->right==NULL) {   //真是叶节点，或者之前做过标记了
            res.push_back(cur->val);
            s.pop();
        }
        if(cur->right != NULL) {    //先放右，同时置为NULL，做标记
            s.push(cur->right);
            cur->right = NULL;
        }
        if(cur->left != NULL) {
            s.push(cur->left);
            cur->left = NULL;
        }
    }
    return res;
}
{% endhighlight %}
**Better solution**

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
    reverse(res.begin(), res.end());    //反转 根右左 变成 左右根
    return res;
}
{% endhighlight %}

### 4. [Binary Tree Level Order Traversal - Leetcode 102](https://leetcode.com/problems/binary-tree-level-order-traversal/)
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

### 5. [Binary Tree Level Order Traversal II - Leetcode 107](https://leetcode.com/problems/binary-tree-level-order-traversal-ii/)
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

### 6. [Binary Tree Zigzag Level Order Traversal - Leetcode 103](https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/)
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
方法2的改编：
{% highlight C++ %}
vector<vector<int> > zigzagLevelOrder(TreeNode *root) {
    vector<vector<int> > res;
    queue<TreeNode *> cur;
    queue<TreeNode *> next;
    if(root == NULL)    return res;
    cur.push(root);
    bool isreversed=false;
    while(!cur.empty()) {
        vector<int> line;
        while(!cur.empty()) {   //两层while，里边处理本行的
            TreeNode * node = cur.front();
            cur.pop();
            line.push_back(node->val);
            if(node->left)   next.push(node->left);
            if(node->right)  next.push(node->right);
        }
        if(isreversed)
            reverse(line.begin(), line.end());    //就多这三行
        isreversed = !isreversed;
        res.push_back(line);
        swap(cur, next);
    }
    return res;
}
{% endhighlight %}
