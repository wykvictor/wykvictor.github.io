---
layout: post
title:  "Binary Tree 3 - Construct Binary Tree"
date:   2016-03-25 14:30:00
tags: [algorithm, leetcode, binary tree, Construct Binary Tree]
categories: Algorithm
---

### 1. [Construct Binary Tree from Preorder and Inorder Traversal - Leetcode 105](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)
Given preorder and inorder traversal of a tree, construct the binary tree.
{% highlight C++ %}
TreeNode *buildTree(vector<int> &preorder, vector<int> &inorder) {
    if(preorder.size() == 0 || inorder.size() == 0)
        return NULL;
    return buildTreeCore(preorder, inorder, 0, preorder.size()-1, 0, inorder.size()-1);
}
TreeNode *buildTreeCore(vector<int> &preorder, vector<int> &inorder, 
                        int preBeg, int preEnd, int inBeg, int inEnd) {
    if(preBeg > preEnd || inBeg > inEnd)
        return NULL;    // 终止条件
    int rootval = preorder[preBeg];
    TreeNode *root = new TreeNode(rootval);    //建立root节点
    int i, len=0;
    // 在in中找root,另，注意len的求法!不等于i
    for(i=inBeg; inorder[i] != rootval && i <= inEnd; i++)
        len++;
    if(i > inEnd) return NULL;    //输入错误，无法建立树
    // 把这些index搞对
    root->left = buildTreeCore(preorder, inorder, preBeg+1, preBeg+len, inBeg, inBeg+len-1);
    root->right = buildTreeCore(preorder, inorder, preBeg+len+1, preEnd, inBeg+len+1, inEnd);
    return root;
}
{% endhighlight %}

### 2. [Construct Binary Tree from Inorder and Postorder Traversal - Leetcode 106](https://leetcode.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)
{% highlight C++ %}
TreeNode *buildTree(vector<int> &inorder, vector<int> &postorder) {
    if(postorder.size() == 0 || inorder.size() == 0)
        return NULL;
    return buildTreeCore(inorder, postorder, 0, inorder.size()-1, 0, postorder.size()-1);
}
TreeNode *buildTreeCore(vector<int> &inorder, vector<int> &postorder,
                        int inBeg, int inEnd, int postBeg, int postEnd) {
    if(postBeg > postEnd || inBeg > inEnd)
        return NULL;    //终止条件
    int rootval = postorder[postEnd];   //主要就这3行不同
    TreeNode *root = new TreeNode(rootval);    //建立root节点
    int i, len=0;
    for(i=inBeg; inorder[i] != rootval && i <= inEnd; i++) //在in中找root,另，注意len的求法!不等于i
        len++;
    if(i > inEnd) return NULL;    //输入错误，无法建立树
  
    root->left = buildTreeCore(inorder, postorder, inBeg, inBeg+len-1, postBeg, postBeg+len-1); //把这些搞对   
    root->right = buildTreeCore(inorder, postorder, inBeg+len+1, inEnd, postBeg+len, postEnd-1);
    return root;
}
{% endhighlight %}

### 3. Serialization/Deserialization of a Binary Tree
```
Assume we have a binary tree below:
    _30_ 
    /    \    
  10    20
  /      /  \ 
50   45  35
Using pre-order traversal, the algorithm should write the following to a file:
30 10 50 # # # 20 45 # # 35 # #
```
{% highlight C++ %}
void writeBinaryTree(BinaryTree *p, ostream &out) {
  if (!p) {
    out << "# ";
  } else {
    out << p->data << " ";
    writeBinaryTree(p->left, out);
    writeBinaryTree(p->right, out);
  }
}
{% endhighlight %}
Deserializing a Binary Tree:
Reading the binary tree from the file is similar. 

We read tokens one at a time using pre-order traversal.

If the token is a sentinel, we ignore it.

If the token is a number, we insert it to the current node, and traverse to its left child, then its right child.
{% highlight C++ %}
void readBinaryTree(BinaryTree *&p, ifstream &fin) {
  int token;
  bool isNumber;
  if (!readNextToken(token, fin, isNumber))
    return;
  if (isNumber) {
    p = new BinaryTree(token);
    readBinaryTree(p->left, fin);
    readBinaryTree(p->right, fin);
  }
}
{% endhighlight %}

### 4. 已知前序遍历，后续遍历，求能够构造多少种二叉树 - Hard
[Solution](http://xinpeng.me/?p=767)

采用递归的思路，前序序列的第一个节点和后续遍历的最后一个节点必然为相同的根节点，否则出错

然后枚举左子树的各种长度，看能否构造出相应的右子树

![preorderpostorder](http://7xno5y.com1.z0.glb.clouddn.com/preorderpostorder.png)
{% highlight C++ %}
int count(vector<int>& preorder, int s1, vector<int>& postorder, int s2, int len) {
    if (len == 0) {
        return 1;
    }
    if(preorder[s1] != postorder[s2+len-1]) {
        return 0;
    }
    int res = 0;
    for (int i = 0; i < len; i++) { // 左子树的各种长度
        int left = count(preorder, s1 + 1, postorder, s2, i);
        int right = count(preorder, s1 + i + 1, postorder, s2 + i, len - i - 1);
        res += left * right;
    }
    return res;
}
int countValidTrees(vector<int>& preorder, vector<int>& postorder) {
    if(preorder.size() != postorder.size())
        return 0;
    return count(preorder, 0, postorder, 0, preorder.size());
}
{% endhighlight %}
