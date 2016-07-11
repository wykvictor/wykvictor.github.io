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
// 在root为根的二叉树中找A,B的LCA:
// 如果找到了就返回这个LCA
// 如果只碰到A，就返回A
// 如果只碰到B，就返回B
// 如果都没有，就返回null
TreeNode *lowestCommonAncestor(TreeNode *root, TreeNode *A, TreeNode *B) {
  if(root == NULL || root == A || root == B) {
    return root;  // 如果是A是B的父节点这种情况，返回A即可
  }
  TreeNode *left = lowestCommonAncestor(root->left, A, B);
  TreeNode *right = lowestCommonAncestor(root->right, A, B);
  if(left != NULL && right != NULL) {
    return root;  // 左右各有1个点，找到了，返回这个LCA; 并且可以一直返回到根root
  }
  if(left == NULL && right == NULL) {
    return NULL;  // 左右都没有
  }
  return left ? left : right;  // 只碰到了A，B中的某一个，返回它
}
{% endhighlight %}

### 2. [Lowest Common Ancestor II](http://www.lintcode.com/en/problem/lowest-common-ancestor-ii/)
```
The node has an extra attribute parent which point to the father of itself. The root's parent is null.
```

{% highlight C++ %}
ParentTreeNode *lowestCommonAncestorII(ParentTreeNode *root,
                                     ParentTreeNode *A,
                                     ParentTreeNode *B) {
  if(root == NULL || root == A || root == B)
      return root;
  stack<ParentTreeNode*> pathA;
  ParentTreeNode *node = A;
  while(node != NULL) {
    pathA.push(node);
    node = node->parent;
  }
  stack<ParentTreeNode*> pathB;
  node = B;
  while(node != NULL) {
    pathB.push(node);
    node = node->parent;
  }
  ParentTreeNode *last;
  while(!pathA.empty() && !pathB.empty()) {
    if(pathA.top() != pathB.top()) {
      return last;
    }
    last = pathA.top();
    pathA.pop();
    pathB.pop();
  }
  return last;  // 题目规定肯定有答案，跳出了，说明A是B的父亲
}
{% endhighlight %}
