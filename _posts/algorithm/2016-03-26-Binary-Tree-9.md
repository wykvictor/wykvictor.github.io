---
layout: post
title:  "Binary Tree 8 - Others"
date:   2016-03-26 12:30:00
tags: [algorithm, leetcode, binary tree]
categories: Algorithm
---

### 1. [Lowest Common Ancestor](http://www.lintcode.com/en/problem/lowest-common-ancestor/)
```
Given the root and two nodes in a Binary Tree. Find the lowest common ancestor(LCA) of the two nodes.
```

方法1，找到2点的path，再遍历2条path找公共节点
{% highlight C++ %}
a) 二叉树拓展到了普通数中，根节点到它的路径 这个O(n) Space的方案
bool GetNodePath(TreeNode* pRoot, TreeNode* pNode, list<TreeNode*>& path){
  if(pRoot == pNode)
    return true;
  path.push_back(pRoot);
  bool found = false;
  
  vector<TreeNode*>::iterator i = pRoot->m_vChildren.begin();
  while(!found && i < pRoot->m_vChildren.end()) { //一找到就跳出
    found = GetNodePath(*i, pNode, path);
    ++i;        //扫描所有的子节点
  }
  if(!found)
    path.pop_back();    //注意!弹出
  return found;   //返回是否能找到, 即有没有该节点
}
b) 求2个链表的，公共节点
TreeNode* GetLastCommonNode(const list<TreeNode*>& path1, const list<TreeNode*>& path2){
  list<TreeNode*>::const_iterator iterator1 = path1.begin();
  list<TreeNode*>::const_iterator iterator2 = path2.begin();
  TreeNode* pLast = NULL;
  
  while(iterator1 != path1.end() && iterator2 != path2.end()){
      if(*iterator1 == *iterator2)
          pLast = *iterator1;  //*iter是<>里的内容，即treenode*
      else
          break;
      iterator1++;
      iterator2++;
  }
  return pLast;
}
c) 上述2个函数，也可以用到求树中2个节点的距离：先求最低公共祖先，然后根据2个list求距离
{% endhighlight %}
方法2，O(1) Space的方案
{% highlight C++ %}

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

