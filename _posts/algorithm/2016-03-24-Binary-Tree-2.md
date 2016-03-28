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
    findKnn(node->left, k, target, out);     //先扫左边的：相当于一个大的滑动窗口，所以用queue实现
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

{% highlight C++ %}
bool isValidBST(TreeNode *root) {
    return isValidBSTCore(root, INT_MIN, INT_MAX);
}
bool isValidBSTCore(TreeNode *root, int minval, int maxval) {  //不需要加&，从上往下传
    if(root == NULL)    //注意空的情况
        return true;
    if(root->val <= minval || root->val >= maxval)  //包括 ==
        return false;
    return isValidBSTCore(root->left, minval, root->val) 
            && isValidBSTCore(root->right, root->val, maxval);
}
{% endhighlight %}
迭代的方案： 从中序出发 --> 也可以用中序递归写法啊，简单 bool isValidBST(TreeNode *root, int &prevVal)
{% highlight C++ %}
bool isValidBST(TreeNode *root) {
    //迭代法：中序遍历，始终记录上一个值
    int prevVal = INT_MIN;
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
A solution using O(n) space is pretty straight forward. Could you devise a constant space solution?
```
{% highlight C++ %}
void recoverTree(TreeNode *root) {
    // O(n)空间，中序遍历，用一个vector保存所有的节点值==》转化问题为 在数组中找逆序的一对数字
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
空间O（1）的方法： - 难
{% highlight C++ %}
void recoverTree(TreeNode *root) {
    // O(1)空间的话，就是在中序遍历的时候，直接保存逆序的2个数字
    TreeNode *left=NULL, *right=NULL, *pre=NULL;
    inTraverse(root, pre, left, right); //找到这样的两个点
    swap(left->val, right->val);
}
void inTraverse(TreeNode *node, TreeNode *&pre, TreeNode *&left, TreeNode *&right) { //取地址！才能返回
    if(node == NULL)
        return;
    inTraverse(node->left, pre, left, right);   //就这 2行中间有变化
    if(pre != NULL && pre->val > node->val){
        if(left == NULL)
            left = pre;     //left的话，是pre有问题!!
        right = node;   //right的话，是node，而且一直在更新!!!一直到真正找到第二个!
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
    root->right = sortedListToBSTCore(pToMid->next, size - mid - 1); //从pToMid->next开始，注意计算好 -1
    return root;
}
{% endhighlight %}
自底向上建树法，难，不考虑掌握
{% highlight C++ %}
TreeNode *sortedListToBST(ListNode *head) {
    //由于链表不能随机读取，因而用上题的解决方案不是很便捷，这里考虑从底向上建树 timeO(n) spaceO(logn)
    int len = 0;
    ListNode *p = head;
    while(p) {
        len++;
        p = p->next;
    }
    return sortedListToBST_aux(head, 0, len - 1);
}
TreeNode *sortedListToBST_aux(ListNode *&root, int begin, int end) {    //注意，&！！！
    if(begin > end)
        return NULL;
    int mid = (begin + end) / 2;
    TreeNode *leftNode = sortedListToBST_aux(root, begin, mid - 1); //注意，这几句的顺序，root被带回新的节点值了!
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
Describe an algorithm to save a Binary Search Tree (BST) to a file in terms of run-time and disk space complexity. You must be able to restore to the exact original BST using the saved format.
Assume we have the following BST:
   _30_ 
   /    \    
  20    40
 /     /  \
10    35  50
Pre-order traversal:
Pre-order traversal is the perfect algorithm for making a copy of a BST. The output of a pre-order traversal of the BST above is 30 20 10 40 35 50. Please note the following important observation:
A node’s parent is always output before itself.
Therefore, when we read the BST back from the file, we are always able to create the parent node before creating its child nodes. The code for writing a BST to a file is exactly the same as pre-order traversal.
30 20 10 40 35 50.
We pass the valid range of the values from the parent node to its child nodes. When we are about to insert a node, we will check if the insert value is in the valid range. If it is, this is the right space to insert. If it is not, we will try the next empty space. Reconstructing the whole BST from a file will take only O(n) time.
```

{% highlight C++ %}
void readBSTHelper(int min, int max, int &insertVal,
                   BinaryTree *&p, ifstream &fin) {
  if (insertVal > min && insertVal < max) {
    int val = insertVal;
    p = new BinaryTree(val);
    if (fin >> insertVal) {
      readBSTHelper(min, val, insertVal, p->left, fin);    //一直读到10，然后发现40不合适，就一直返回，直到执行到
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
    \       /     /      / \      \
     3    2    1     1   3     2
    /     /       \                 \
   2    1        2                3
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
    \       /     /      / \      \
     3    2    1     1   3     2
    /     /       \                 \
   2    1        2                3
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
        for(int j=0; j<leftTree.size(); j++) { //左右子树，排列组合起来构成结果
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
