---
layout: post
title:  "Binary Tree 9 - Others"
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

### 3. [Complete Binary Tree](http://www.lintcode.com/en/problem/complete-binary-tree/)
```
Check a binary tree is completed or not. A complete binary tree is a binary tree that every level is completed filled except the deepest level. In the deepest level, all nodes must be as left as possible. 
```

ResultType方法
{% highlight C++ %}
struct Result {
  int height;
  bool iscomplete;
  bool isfull;
  Result(int i = -1, bool is = false, bool full = false)
      : height(i), iscomplete(is), isfull(full) {}
};
Result isCompleteHelper(TreeNode* root) {
  if (root == NULL) {
    return Result(0, true, true);
  }
  Result left = isCompleteHelper(root->left);
  Result right = isCompleteHelper(root->right);
  if (left.iscomplete == false || right.iscomplete == false) {
    return Result();  // false
  }
  /* // do not need
  if(left.height < right.height || left.height - right.height > 1) {
      return Result();
  }*/
  // 相等,left is Full, right is complete
  if (left.height == right.height) {
    if (left.isfull && right.iscomplete) {
      return Result(left.height + 1, true, right.isfull);
    }
    return Result();
  }
  // left is complete, right is full
  if (left.height == right.height + 1) {
    if (left.iscomplete && right.isfull) {
      return Result(left.height + 1, true, false);
    }
    return Result();
  }
  return Result();
}
bool isComplete(TreeNode* root) {
  if (root == NULL) return true;
  return isCompleteHelper(root).iscomplete;
}
{% endhighlight %}
推荐方法，层序遍历，叶节点填洞，最后的结尾都是洞
{% highlight C++ %}
bool isComplete(TreeNode* root) {
  if (root == NULL) return true;
  vector<TreeNode*> q;  // use vector, in order to index it
  q.push_back(root);
  for (int i = 0; i < q.size(); i++) {
    TreeNode* node = q[i];
    if (node == NULL) {
      continue;
    }
    q.push_back(node->left);
    q.push_back(node->right);
  }
  int lastNotNULL = -1;
  for (int i = q.size() - 1; i >= 0; i--) {
    if (q[i] != NULL) {
      lastNotNULL = i;
      break;
    }
  }
  for (int i = lastNotNULL - 1; i > 0; i--) {
    if (q[i] == NULL) {
      return false;
    }
  }
  return true;
}
{% endhighlight %}

### 4. [Binary Tree Longest Consecutive Sequence](http://www.lintcode.com/en/problem/binary-tree-longest-consecutive-sequence/)
```
Given a binary tree, find the length of the longest consecutive sequence path.
The longest consecutive path need to be from parent to child (cannot be the reverse).
```

DFS - Traverse:
{% highlight C++ %}
// 好理解，边Traverse，边记录
void dfs(TreeNode* root, int& result, int curLen, int lastNum) {
  if (root == NULL) {
    return;
  }
  if (root->val == lastNum + 1)
    curLen++;
  else
    curLen = 1;  // 不连续，归位
  if (curLen > result) result = curLen;
  dfs(root->left, result, curLen, root->val);
  dfs(root->right, result, curLen, root->val);
}
int longestConsecutive(TreeNode* root) {
  int result = -1;
  dfs(root, result, 0, 0);
  return result;
}
{% endhighlight %}
DFS - Divide & Conquer:
{% highlight C++ %}
// 不太好理解：dfs函数代表，加上左/右子树后，返回最长值
int dfs(TreeNode* root, int curLen, int lastNum) {
  if (root == NULL) {
    return 0;
  }
  if (root->val == lastNum + 1)
    curLen++;
  else
    curLen = 1;  // 不连续，归位
  // 区别：
  int leftLongest = dfs(root->left, curLen, root->val);
  int rightLongest = dfs(root->right, curLen, root->val);
  return max(curLen, max(leftLongest, rightLongest));
}
int longestConsecutive(TreeNode* root) { return dfs(root, 0, 0); }
{% endhighlight %}

### 5. [Binary Tree Paths](http://www.lintcode.com/en/problem/binary-tree-paths/)
{% highlight C++ %}
void dfs(TreeNode* root, vector<string>& res, vector<int>& line) {
  if (root->left == NULL && root->right == NULL) {
    string tmp;
    for (int i = 0; i < line.size(); i++) {
      tmp += to_string(line[i]) + "->";
    }
    tmp += to_string(root->val);  // 最後一個直接加上，不放line了
    res.push_back(tmp);
    return;
  }
  line.push_back(root->val);
  if (root->left != NULL) dfs(root->left, res, line);
  if (root->right != NULL) dfs(root->right, res, line);
  line.pop_back();
}
vector<string> binaryTreePaths(TreeNode* root) {
  vector<string> res;
  if (root == NULL) return res;
  vector<int> line;
  dfs(root, res, line);
  return res;
}
// 或直接用string，简洁!
void dfs(TreeNode* root, vector<string>& res, string line) {  // string无&，拷贝
  if (root->left == NULL && root->right == NULL) {
    res.push_back(line);
    return;
  }
  if (root->left != NULL)
    dfs(root->left, res, line + "->" + to_string(root->left->val));
  if (root->right != NULL)
    dfs(root->right, res, line + "->" + to_string(root->right->val));
}
vector<string> binaryTreePaths(TreeNode* root) {
  vector<string> res;
  if (root == NULL) return res;
  dfs(root, res, to_string(root->val));
  return res;
}
{% endhighlight %}
