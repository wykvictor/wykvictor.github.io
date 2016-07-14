---
layout: post
title:  "Binary Tree 2 - Binary Search Tree"
date:   2016-03-24 14:30:00
tags: [algorithm, leetcode, binary tree, Binary Search Tree]
categories: Algorithm
---

### 1. Find closest value of a BST
Give a BST, find a node whose value is closest to the target.
{% highlight C++ %}
TreeNode* closestBST(TreeNode* root, int val){ 
    if(!root) return NULL; 
    if(root->val == val) return root; 
    if(val < root->val){ 
        if(!root->left) return root; 
        TreeNode * p = closestBST(root->left, val); 
        return abs(p->val - val) > abs(root->val - val) ? root : p; 
    }else{ 
        if(!root->right) return root; 
        TreeNode * p = closestBST(root->right, val); 
        return abs(p->val - val) > abs(root->val - val) ? root : p; 
    }    
    return NULL; 
}
{% endhighlight %}

### 2. Find k nearest neightbours in BST - 好题，中序遍历
{% highlight C++ %}
void findKnn(TreeNode* node, int k, int target, deque<int>& out){ 
    if (!node) return;
    // 先扫左边的：相当于一个大的滑动窗口，所以用queue实现
    findKnn(node->left, k, target, out);
    if (out.size() < k) 
        out.push_back(node->val); 
    else if(out.size() == k){ 
        if (abs(target-node->val)<abs(target-out.front())){ 
            out.pop_front(); 
            out.push_back(node->val);   //替换出更合适的
        } else
            return;    //这样可以退出了吧，不用继续了！
    } 
    findKnn(node->right, k, target, out);  //再弄右边的
} 
{% endhighlight %}

### 3. [Validate BST - Leetcode 98](https://leetcode.com/problems/validate-binary-search-tree/)
```
Given a binary tree, determine if it is a valid binary search tree (BST).
Assume a BST is defined as follows:
The left subtree of a node contains only nodes with keys less than the node's key.
The right subtree of a node contains only nodes with keys greater than the node's key.
Both the left and right subtrees must also be binary search trees
```

错误解法：需要得到右子树的最小值,只比较right不对
{% highlight C++ %}
// wrong case: {10,5,15,#,#,6,20}
bool isValidBST(TreeNode *root) {
  if(root == NULL) {
    return true;
  }
  if(root->left && root->left->val >= root->val)
    return false;
  if(root->right && root->right->val <= root->val)
    return false;
  return isValidBST(root->left) && isValidBST(root->right);
}
{% endhighlight %}
先序，递归，core函数较难想到, traverse方法：
{% highlight C++ %}
bool isValidBST(TreeNode *root) {
  // 若测试用例有INT_MIN节点，则此方法不可行！
  return isValidBSTCore(root, INT_MIN, INT_MAX);
}
// 不需要加&，从上往下传
bool isValidBSTCore(TreeNode *root, int minval, int maxval) {
  if(root == NULL)    //注意空的情况
    return true;
  if(root->val <= minval || root->val >= maxval)  //包括 ==
    return false;
  return isValidBSTCore(root->left, minval, root->val) 
        && isValidBSTCore(root->right, root->val, maxval);
}
{% endhighlight %}
也可以用中序递归写法，推荐方法，traverse方法：
{% highlight C++ %}
bool isValidBSTHelper(TreeNode *root, TreeNode * &prev) {
  if(root == NULL) {
    return true;
  }
  if(!isValidBSTHelper(root->left, prev))
    return false;
  if(prev != NULL &&　prev->val >= root->val) {
    //cout << prev->val << " " << root->val << endl;
    return false;
  }
  prev = root;
  return isValidBSTHelper(root->right, prev);
}
bool isValidBST(TreeNode *root) {
  TreeNode *prev = NULL; //!不能直接传入NULL，需要声明这个4字节的指针，然后才能取引用
  return isValidBSTHelper(root, prev);
}
{% endhighlight %}
迭代的方案： 中序
{% highlight C++ %}
bool isValidBST(TreeNode *root) {
  //迭代法：中序遍历，始终记录上一个值
  int prevVal = INT_MIN;  // also wrong: need use treenode to record
  stack<TreeNode*> s;
  TreeNode *node = root;
  while(!s.empty() || node) {
    if(node) {
      s.push(node);
      node = node->left;
    }else { //此时访问
      node = s.top();
      s.pop();
      //就这2行不同
      if(prevVal >= node->val)
          return false;
      prevVal = node->val;
      node = node->right; //转向右子树
    }
  }
  return true;
}
{% endhighlight %}

### 4. [Recover BST - Leetcode 99](https://leetcode.com/problems/recover-binary-search-tree/)
```
Two elements of a binary search tree (BST) are swapped by mistake.
Recover the tree without changing its structure.
Note:
A solution using O(n) space is pretty straight forward.
Could you devise a constant space solution?
```
{% highlight C++ %}
void recoverTree(TreeNode *root) {
    // O(n)空间，中序遍历，用一个vector保存所有的节点值
    // ==》转化问题为 在数组中找逆序的一对数字
    vector<TreeNode*> index;
    inTraverse(root, index);
    int left=0, right=index.size()-1;
    for(;left < right; left++)  //从前往后，找第一个逆序
        if(index[left]->val > index[left+1]->val)
            break;
    for(;right > left; right--)   //从后往前，找第二个逆序
        if(index[right]->val < index[right-1]->val)
            break;
    swap(index[left]->val, index[right]->val);
}
void inTraverse(TreeNode* node, vector<TreeNode*> &index) {
    if(node == NULL)
        return;
    inTraverse(node->left, index);
    index.push_back(node);
    inTraverse(node->right, index);
}
{% endhighlight %}
空间O（1）的方法：- Hard
{% highlight C++ %}
void recoverTree(TreeNode *root) {
    // O(1)空间的话，就是在中序遍历的时候，直接保存逆序的2个数字
    TreeNode *left=NULL, *right=NULL, *pre=NULL;
    inTraverse(root, pre, left, right); //找到这样的两个点
    swap(left->val, right->val);
}
void inTraverse(TreeNode *node, TreeNode *&pre, TreeNode *&left, TreeNode *&right) { //取地址&才能返回
    if(node == NULL)
        return;
    inTraverse(node->left, pre, left, right);   //就这2行中间有变化
    if(pre != NULL && pre->val > node->val){
        if(left == NULL)
            left = pre;     //left的话，是pre有问题!!
        //right的话，是node，立马赋值为node，但之后很可能会更新，一直真正找到!
        right = node;
    }
    pre = node;
    inTraverse(node->right, pre, left, right);
}
{% endhighlight %}

### 5. [Convert Sorted Array to Binary Search Tree - Leetcode 108](https://leetcode.com/problems/convert-sorted-array-to-binary-search-tree/)
Given an array where elements are sorted in ascending order, convert it to a height balanced BST.
{% highlight C++ %}
TreeNode *sortedArrayToBST(vector<int> &num) {
    //从中间划分，也是递归的思路
    int size = num.size();
    return sortedArrayToBSTCore(num, 0, size-1);
}
TreeNode *sortedArrayToBSTCore(vector<int> &num, int start, int end) {
    if(start > end)     return NULL;
    int mid = start + ((end-start)>>1);
    TreeNode *node = new TreeNode(num[mid]);
    //if(start == end)    return node;  下边也包括了，不需要单独来
    node->left = sortedArrayToBSTCore(num, start, mid-1);
    node->right = sortedArrayToBSTCore(num, mid+1, end);
    return node;    
}
{% endhighlight %}

### 6. [Convert Sorted List to Binary Search Tree - Leetcode 109](https://leetcode.com/problems/convert-sorted-list-to-binary-search-tree/)
Given a singly linked list where elements are sorted in ascending order, convert it to a height balanced BST.
{% highlight C++ %}
TreeNode *sortedListToBST(ListNode *head) {
    //首先模拟上题的解法
    if(head == NULL)    return NULL;
    ListNode *p = head;
    int size = 1;
    while(p->next != NULL) {
        size++;
        p = p->next;
    }
    return sortedListToBSTCore(head, size);
}
TreeNode *sortedListToBSTCore(ListNode *head, int size) {   //有几个元素
    if(size <= 0)        //有"="，0代表没有节点，直接返回NULL
        return NULL;
    int mid = size >> 1;
    int step = 0;
    ListNode *pToMid=head;
    while(step++ < mid){
        pToMid = pToMid->next;  //找到中间的节点
    }
    TreeNode *root = new TreeNode(pToMid->val);    //初始化
    root->left = sortedListToBSTCore(head, mid);   //前一半是 mid 的长度
    // 从pToMid->next开始，注意计算好-1
    root->right = sortedListToBSTCore(pToMid->next, size - mid - 1);
    return root;
}
{% endhighlight %}
自底向上建树法，very hard
{% highlight C++ %}
TreeNode *sortedListToBST(ListNode *head) {
    //链表不能随机读取，用上题的解决方案不便捷，考虑从底向上建树timeO(n),spaceO(logn)
    int len = 0;
    ListNode *p = head;
    while(p) {
        len++;
        p = p->next;
    }
    return sortedListToBST_aux(head, 0, len - 1);
}
//注意! &
TreeNode *sortedListToBST_aux(ListNode *&root, int begin, int end) {
    if(begin > end)
        return NULL;
    int mid = (begin + end) / 2;
    // 注意，这几句的顺序，root被带回新的节点值了!
    TreeNode *leftNode = sortedListToBST_aux(root, begin, mid - 1);
    TreeNode *parent = new TreeNode(root->val);
    parent->left = leftNode;
    root = root->next;
    parent->right = sortedListToBST_aux(root, mid + 1, end);
    return parent;
}
{% endhighlight %}

### 7. Saving a Binary Search Tree to a File
```
和前序，中序建树类似。但该方法简洁些
Describe an algorithm to save a Binary Search Tree (BST) to a file in terms of run-time and disk space complexity.
You must be able to restore to the exact original BST using the saved format.
Assume we have the following BST:
   _30_ 
   /    \    
  20    40
 /     /  \
10    35  50
Pre-order traversal:
Pre-order traversal is the perfect algorithm for making a copy of a BST.
The output of a pre-order traversal of the BST above is 30 20 10 40 35 50.

Please note the following important observation:
A node’s parent is always output before itself.
Therefore, when we read the BST back from the file, we are always able to create the parent node before creating its child nodes.
The code for writing a BST to a file is exactly the same as pre-order traversal.
30 20 10 40 35 50.

We pass the valid range of the values from the parent node to its child nodes.
When we are about to insert a node, we will check if the insert value is in the valid range.
If it is, this is the right space to insert.
If it is not, we will try the next empty space.
Reconstructing the whole BST from a file will take only O(n) time.
```

{% highlight C++ %}
void readBSTHelper(int min, int max, int &insertVal,
                   BinaryTree *&p, ifstream &fin) {
  if (insertVal > min && insertVal < max) {
    int val = insertVal;
    p = new BinaryTree(val);
    if (fin >> insertVal) {
      // 一直读到10，然后发现40不合适，就一直返回，直到执行到
      readBSTHelper(min, val, insertVal, p->left, fin);
      readBSTHelper(val, max, insertVal, p->right, fin);
    }
  }
}
void readBST(BinaryTree *&root, ifstream &fin) {
  int val;
  fin >> val;
  readBSTHelper(INT_MIN, INT_MAX, val, root, fin);
}
{% endhighlight %}

### 8. Build double linked list from BST
二叉搜索树转换为排序的双向链表
{% highlight C++ %}
空间O(1): 用了中序的模板
DoubleList *head = NULL; 
DoubleList *end = NULL; 
DoubleList* ConverToDoubleList(Node *root){ 
    Node *temp = root; 
    if(temp == NULL) return NULL; //1，返回
    ConverToDoubleList(temp->left);     //2，处理左边的
    //3，中间的步骤
    temp->left = end; 
    if (end == NULL)     //若是开头节点
        end = head = temp; 
    else 
        end->right = temp; 
    end = temp; 
    ConverToDoubleList(temp->right);     //搞右边的
    return head; 
}
{% endhighlight %}

### 9. [Unique Binary Search Trees - Leetcode 96](https://leetcode.com/problems/unique-binary-search-trees/)
```
Given n, how many structurally unique BST's (binary search trees) that store values 1...n?
For example,
Given n = 3, there are a total of 5 unique BST's.
   1        3    3     2     1
    \      /    /     / \     \
     3    2    1     1   3     2
    /    /      \               \
   2    1        2               3
```

{% highlight C++ %}
int numTrees(int n) {
    //DP pdf中有透彻分析!! 
    //即，从1到n轮流为根，每种情况的数目=左子树数目*右子树数目(排列组合嘛)
    vector<int> f(n+1, 0); //从0到n 先都初始化为0!
    f[0] = f[1] = 1;    //注意f(0)和f(1)都是1种，空树也是一种!
    for(int i=2; i<=n; i++) {    //从第2个元素开始计算
        for(int j=1; j<=i; j++) //从1到i轮流为根元素  相当于求和一直
            f[i] += f[j-1] * f[i-j];
    }
    return f[n];
}
{% endhighlight %}

### 10. [Unique Binary Search Trees II - Leetcode 95 Hard](https://leetcode.com/problems/unique-binary-search-trees-ii/)
```
Given n, generate all structurally unique BST's (binary search trees) that store values 1...n.
For example,
Given n = 3, your program should return all 5 unique BST's shown below.
   1        3    3     2     1
    \      /    /     / \     \
     3    2    1     1   3     2
    /    /      \               \
   2    1        2               3
```

{% highlight C++ %}
vector<TreeNode *> generateTrees(int n) {
    return generateTreesCore(1,n);
}
vector<TreeNode *> generateTreesCore(int left, int right) {
    vector<TreeNode *> res;
    if(left > right) {    //递归，收敛条件
        res.push_back(NULL);  //压进去一个NULL的tree
        return res;
    }
    //开始遍历，从left到right，分别当头结点
    for(int i=left; i<=right; i++) {
        vector<TreeNode *> leftTree = generateTreesCore(left, i-1);
        vector<TreeNode *> rightTree = generateTreesCore(i+1, right);
        // 左右子树，排列组合起来构成结果
        for(int j=0; j<leftTree.size(); j++) {
            for(int k=0; k<rightTree.size(); k++){
                TreeNode *head = new TreeNode(i);   //头
                head->left = leftTree[j];
                head->right = rightTree[k];
                res.push_back(head);
            }
        }
    }
    return res;
}
{% endhighlight %}

### 11. [Insert Node in a BST](http://www.lintcode.com/en/problem/insert-node-in-a-binary-search-tree/)
一直找到最边上，也不用交换节点啥的，就在尾部连接, 递归法
{% highlight C++ %}
TreeNode* insertNode(TreeNode* root, TreeNode* node) {
  if(root == NULL || node == NULL)
    return node;  // if root is {}, return {node}
  if(root->val < node->val) {
    if(root->right == NULL)
      root->right = node;
    else
      insertNode(root->right, node);
  } else {
    if(root->left == NULL)
      root->left = node;
    else
      insertNode(root->left, node);
  }
  return root;
}
{% endhighlight %}
迭代：
{% highlight C++ %}
TreeNode* insertNode(TreeNode* root, TreeNode* node) {
  // write your code here
  if(root == NULL || node == NULL)
    return node;
  TreeNode* p = root;
  while(p != NULL) {
    if(p->val < node->val) {
      if(p->right == NULL) {
        p->right = node;
        break;
      }
      p = p->right;
    } else {
      if(p->left == NULL) {
        p->left = node;
        break;
      }
      p = p->left;
    }
  }
  return root;
}
{% endhighlight %}

### 12. [Remove Node in a BST - Hard](http://www.lintcode.com/en/problem/remove-node-in-binary-search-tree/)
```
Given a root of Binary Search Tree with unique value for each node.
Remove the node with given value.
If there is no such a node with given value, do nothing. 
```

[solution-1](http://www.mathcs.emory.edu/~cheung/Courses/171/Syllabus/9-BinTree/BST-delete.html)
[solution-2](http://www.mathcs.emory.edu/~cheung/Courses/171/Syllabus/9-BinTree/BST-delete2.html)
{% highlight C++ %}
TreeNode* removeNode(TreeNode* root, int value) {
    // write your code here
    TreeNode dummyNode(-1);  // in case delete root
    dummyNode.right = root;
    deleteNode(root, value, &dummyNode);
    return dummyNode.right;
}
// helper function
void deleteNode(TreeNode* root, int value, TreeNode* parent) {
    if(root == NULL)  return;
    if(root->val < value) {
        deleteNode(root->right, value, root);
        return;
    }
    if(root->val > value) {
        deleteNode(root->left, value, root);
        return;
    }
    // find and delete
    // case 1: leaf node
    if(root->left==NULL && root->right==NULL) {
        if(parent->left == root) {
            parent->left = NULL;
        } else {
            parent->right = NULL;
        }
        delete root;
        return;
    }
    // case 2: only one child
    if(root->left==NULL || root->right==NULL) {
        if(parent->left == root) {
            parent->left = root->left ? root->left : root->right;
        } else {
            parent->right = root->right ? root->right : root->left;
        }
        delete root;
        return;
    }
    // case 3: 2 children
    // step1: find the smallest of the right
    TreeNode *minNode = root->right;
    while(minNode->left != NULL) {
        parent = minNode;
        minNode = minNode->left;
    }
    // step 2: copy the val
    root->val = minNode->val;
    // step 3: delete it
    parent->left = NULL;
    delete minNode;
}
{% endhighlight %}
合并case 1 and case 2
{% highlight C++ %}
// helper function
void deleteNode(TreeNode* root, int value, TreeNode* parent) {
    if(root == NULL)  return;
    if(root->val < value) {
        deleteNode(root->right, value, root);
        return;
    }
    if(root->val > value) {
        deleteNode(root->left, value, root);
        return;
    }
    // find and delete
    // case 1: leaf node and case 2: only one child
    if(root->left==NULL || root->right==NULL) {
        if(parent->left == root) {
            parent->left = root->left ? root->left : root->right;
        } else {
            parent->right = root->right ? root->right : root->left;
        }
        delete root;
        return;
    }
    // case 3: 2 children
    // step1: find the smallest of the right
    TreeNode *minNode = root->right;
    while(minNode->left != NULL) {
        parent = minNode;
        minNode = minNode->left;
    }
    // step 2: copy the val
    root->val = minNode->val;
    // step 3: delete it
    parent->left = NULL;
    delete minNode;
}
{% endhighlight %}

### 13. [Search Range in Binary Search Tree - Hard](http://www.lintcode.com/en/problem/search-range-in-binary-search-tree/)
```
Given two values k1 and k2 (where k1 < k2) and a root pointer to a BST.
Find all the keys of tree in range k1 to k2. i.e. print all x such that k1<=x<=k2 and x is a key of given BST. Return all the keys in ascending order.
```
{% highlight C++ %}
vector<int> searchRange(TreeNode* root, int k1, int k2) {
  vector<int> res;
  DFS(res, root, k1, k2);
  return res;
}
void DFS(vector<int> &res, TreeNode* root, int k1, int k2) {
  if(root == NULL)  return;
  /*if(root->val > k2 || root->val < k1) {
        return;  // 错误剪枝,return早了
  }*/
  if(root->val >= k1) { //优化，如果超出范围，剪枝！
    DFS(res, root->left, k1, k2);
  }
  if(root->val >= k1 && root->val <= k2) {
    res.push_back(root->val);
  }
  if(root->val <= k2) {
    DFS(res, root->right, k1, k2);
  }
}
{% endhighlight %}

### 14. [Binary Search Tree Iterator](http://www.lintcode.com/en/problem/binary-search-tree-iterator/)
```
next() and hasNext() queries run in O(1) time in average.
Challenge: Extra memory usage O(h), h is the height of the tree.
Super Star: Extra memory usage O(1)
```
{% highlight C++ %}
/**
 * Example of iterate a tree:
 * BSTIterator iterator = BSTIterator(root);
 * while (iterator.hasNext()) {
 *    TreeNode * node = iterator.next();
 *    do something for node
 */
class BSTIterator {
public:
    stack<TreeNode*> s;
    TreeNode *cur;
    //@param root: The root of binary tree.
    BSTIterator(TreeNode *root): cur(root) {
        /*while(cur != NULL) { //init to left-most
            s.push(cur);
            cur = cur->left;
        }*/ //comment out: next() has the same function
    }
    //@return: True if there has next node, or false
    bool hasNext() {
        return (cur != NULL) || (!s.empty());
    }
    //@return: return next node
    TreeNode* next() {
        // find cur->next before return cur
        while(cur != NULL) { //go to left-most
            s.push(cur);
            cur = cur->left;
        }
        TreeNode* nxt = s.top();
        s.pop();
        cur = nxt->right;
        return nxt;
    }
};
{% endhighlight %}

### 15. [Inorder Successor in Binary Search Tree](http://www.lintcode.com/en/problem/inorder-successor-in-binary-search-tree/)
```
Given a binary search tree and a node in it, find the in-order successor of that node in the BST.
If the given node has no in-order successor in the tree, return null.
It's guaranteed p is one node in the given tree.

Time Complexity: O(h), where h is the height of the BST.
```
{% highlight C++ %}
// 错误解法：2,1这种树，1的后继为2，是它的父节点，所以递归写法拿不到父节点，不可行
TreeNode* inorderSuccessor(TreeNode* root, TreeNode* p) {
  if(root == NULL)  return NULL;
  if(p == root) {
    TreeNode* successor = root->right;
    while(successor->left != NULL) {
      successor = successor->left;
    }
    return successor;
  }
  if(p->val > root->val) {
    return inorderSuccessor(root->right, p);
  } else {
    return inorderSuccessor(root->left, p);
  }
}
// 正确：迭代，记录父节点
TreeNode* inorderSuccessor(TreeNode* root, TreeNode* p) {
  if(root == NULL)  return NULL;
  TreeNode* successor = NULL;
  // find p
  while(root != p) {
    //successor = root;  // 记录上一个后继:不是这里记！
    if(p->val > root->val) {
      root = root->right;
    } else {
      successor = root;  // 记录上一个后继:这里记！往左走的时候
      root = root->left;
    }
  }
  // if no right child, then return parent
  if(p->right == NULL) {
    return successor;
  }
  // has right child, then find the leftmost child
  successor = root->right;
  while(successor->left != NULL) {
    successor = successor->left;
  }
  return successor;
}
{% endhighlight %}
