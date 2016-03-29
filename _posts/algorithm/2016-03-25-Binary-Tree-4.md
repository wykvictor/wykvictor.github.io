---
layout: post
title:  "Binary Tree 4 - Compare Tree"
date:   2016-03-25 15:30:00
tags: [algorithm, leetcode, binary tree, Construct Binary Tree]
categories: Algorithm
---

### 1. Two sum in a BST - Hard
```
Given a BST and a value x. Find two nodes in the tree whose sum is equal x. Additional space:
O(height of the tree). It is not allowed to modify the tree. 
```

The Brute Force Solution is to consider each pair in BST and check whether the sum equals to X. The time complexity of this solution will be O(n^2).
A Better Solution is to create an auxiliary array and store Inorder traversal of BST in the array. The array will be sorted as Inorder traversal of BST always produces sorted data. Once we have the Inorder traversal, we can pair in O(n) time (See this for details). This solution works in O(n) time, but requires O(n) auxiliary space.
A space optimized solution is discussed in previous post. The idea was to first in-place convert BST to Doubly Linked List (DLL), then find pair in sorted DLL in O(n) time. This solution takes O(n) time and O(Logn) extra space, but it modifies the given BST.
The solution discussed below takes O(n) time, O(Logn) space and doesn’t modify BST. The idea is same as finding the pair in sorted array (See method 1 of this for details). We traverse BST in Normal Inorder and Reverse Inorder simultaneously. In reverse inorder, we start from the rightmost node which is the maximum value node. In normal inorder, we start from the left most node which is minimum value node. We add sum of current nodes in both traversals and compare this sum with given target sum. If the sum is same as target sum, we return true. If the sum is more than target sum, we move to next node in reverse inorder traversal, otherwise we move to next node in normal inorder traversal. If any of the traversals is finished without finding a pair, we return false. Following is C++ implementation of this approach.
最后这种好方法：
{% highlight C++ %}
bool isPairPresent(struct node *root, int target)
{
    struct Stack* s1 = createStack(MAX_SIZE);
    struct Stack* s2 = createStack(MAX_SIZE);
    // Note the sizes of stacks is MAX_SIZE, we can find the tree size and
    // fix stack size as O(Logn) for balanced trees like AVL and Red Black
    // tree. We have used MAX_SIZE to keep the code simple
  
    bool done1 = false, done2 = false;
    int val1 = 0, val2 = 0;
    struct node *curr1 = root, *curr2 = root;
  
    // The loop will break when we either find a pair or one of the two
    // traversals is complete
    while (1){
        while (done1 == false) {    //控制是否做这个循环
            if (curr1 != NULL){
                push(s1, curr1);
                curr1 = curr1->left;
            } else {
                if(isEmpty(s1))
                    done1 = 1;
                else{
                    curr1 = pop(s1);
                    val1 = curr1->val;
                    curr1 = curr1->right;
                    done1 = 1;
                }
            }
        }
        while (done2 == false) {
            if (curr2 != NULL){
                push(s2, curr2);
                curr2 = curr2->right;
            } else {
                if (isEmpty(s2))
                    done2 = 1;
                else{
                    curr2 = pop(s2);
                    val2 = curr2->val;
                    curr2 = curr2->left;
                    done2 = 1;
                }
            }
        }
        if ((val1 != val2) && (val1 + val2) == target) {
            printf("\n Pair Found: %d + %d = %d\n", val1, val2, target);
            return true;
        } else if ((val1 + val2) < target)
            done1 = false;
        else if ((val1 + val2) > target)
            done2 = false;
        if (val1 >= val2)
            return false;
    }
}
{% endhighlight %}

5， Minimum depth of binary tree
Given a binary tree, find its minimum depth.
The minimum depth is the number of nodes along the shortest path from the root node down to the nearest leaf node.
int minDepth(TreeNode *root) {
    if(root == NULL)
        return 0;
    int left = minDepth(root->left);
    int right = minDepth(root->right);
    return (left==0 || right==0) ? max(left, right)+1 : min(left, right)+1;    //只要没有兄弟，就是另一个的+1
}

6， Maximum depth of binary tree
Given a binary tree, find its maximum depth.
The maximum depth is the number of nodes along the longest path from the root node down to the farthest leaf node.
int maxDepth(TreeNode *root) {
    if(root == NULL)
        return 0;
    return max(maxDepth(root->left), maxDepth(root->right))+1;
}

7，Symmetric tree
Given a binary tree, check whether it is a mirror of itself (ie, symmetric around its center).
For example, this binary tree is symmetric:
    1
   / \
  2   2
 / \ / \
3  4 4  3
But the following is not:
    1
   / \
  2   2
   \   \
   3    3
递归版本：
bool isSymmetric(TreeNode *root) {
    //递归版本
    if(root == NULL)
        return true;
    return isSymmetricCore(root->left, root->right);
}
bool isSymmetricCore(TreeNode *leftNode, TreeNode *rightNode) {
    if(leftNode==NULL && rightNode==NULL)   //先处理有个空的这种情况
        return true;
    if(leftNode==NULL || rightNode==NULL)
        return false;
    return (leftNode->val == rightNode->val) && isSymmetricCore(leftNode->left, rightNode->right) 
            && isSymmetricCore(leftNode->right, rightNode->left);
}
迭代：
bool isSymmetric(TreeNode *root) {
    //迭代版本，用队列吧，广度遍历
    if(root == NULL)
        return true;
    if(root->left==NULL && root->right==NULL)
        return true;
    if(root->left==NULL || root->right==NULL)
        return false;
    //下面开始正常处理流程
    queue<TreeNode *> q;
    q.push(root->left);
    q.push(root->right);
    while(!q.empty()) {
        TreeNode *leftNode = q.front();
        q.pop();
        TreeNode *rightNode = q.front();
        q.pop();
        if(leftNode->val != rightNode->val)
            return false;
        if(leftNode->left != NULL) {
            if(rightNode->right == NULL)
                return false;
            q.push(leftNode->left);
            q.push(rightNode->right);
        } else if(rightNode->right != NULL)
            return false;
        if(leftNode->right != NULL) {
            if(rightNode->left == NULL)
                return false;
            q.push(leftNode->right);
            q.push(rightNode->left);
        } else if(rightNode->left != NULL)
            return false;
    }
    return true;    //最终都判断完，才true
}
简化一下逻辑，空的也可以压入queue，就和递归思路一样了，里边就不需要判断4个点了，2个就够了：
bool isSymmetric(TreeNode *root) {
    //迭代版本，用队列吧，广度遍历
    if(root == NULL)
        return true;
    //下面开始正常处理流程
    queue<TreeNode *> q;
    q.push(root->left); //NULL也压入
    q.push(root->right);
    while(!q.empty()) {
        TreeNode *leftNode = q.front();
        q.pop();
        TreeNode *rightNode = q.front();
        q.pop();
        if(leftNode==NULL && rightNode==NULL)
            continue;   //true时，继续判断
        if(leftNode==NULL || rightNode==NULL)   //之后，肯定都不空
            return false;
        if(leftNode->val != rightNode->val)
            return false;
         
        q.push(leftNode->left);
        q.push(rightNode->right);
        q.push(leftNode->right);
        q.push(rightNode->left);
    }
    return true;    //最终都判断完，才true
}

8，Same tree
Given two binary trees, write a function to check if they are equal or not.
Two binary trees are considered equal if they are structurally identical and the nodes have the same value.
bool isSameTree(TreeNode *p, TreeNode *q) {
    if(p == NULL && q == NULL)
        return true;
    if(p == NULL || q == NULL)
        return false;
    if(p->val != q->val)
        return false;
    return isSameTree(p->left, q->left) && isSameTree(p->right, q->right);
}
同样的，遍历方案也可：几乎和上边的一模一样
bool isSameTree(TreeNode *p, TreeNode *q) {
    queue<TreeNode *> qnodes;
    qnodes.push(p);
    qnodes.push(q);
    while(!qnodes.empty()) {
        TreeNode *lnode = qnodes.front();
        qnodes.pop();
        TreeNode *rnode = qnodes.front();
        qnodes.pop();
        if(lnode==NULL && rnode==NULL)
            continue;
        if(lnode==NULL || rnode==NULL)
            return false;
        if(lnode->val != rnode->val)
            return false;
         
        qnodes.push(lnode->left);
        qnodes.push(rnode->left);
        qnodes.push(lnode->right);
        qnodes.push(rnode->right);
    }
    return true;
}

18，Flatten Binary Tree to Linked List - 思路想好较重要
Given a binary tree, flatten it to a linked list in-place.
For example,
Given
         1
        / \
       2   5
      / \   \
     3   4   6
The flattened tree should look like:
   1
    \
     2
      \
       3
        \
         4
          \
           5
            \
             6
递归版1，变左变右，左插右前边 时间复杂度 O(n)，空间复杂度 O(logn)
void flatten(TreeNode *root) {
    //弄成单向链表即可，类似preorder的遍历，递归，root的左右先做这个动作，然后把left插入到root和right之间即可
    if(root == NULL)    return;
    flatten(root->left);
    flatten(root->right);
    if(root->left == NULL)  return; //若left为空，就已经成了，不需要下边的插的步骤
    TreeNode *p = root->left;
    while(p->right != NULL)  //一直找到left的最后一个节点 
        p = p->right;
    p->right = root->right;
    root->right = root->left;
    root->left = NULL;    //必须封口，不能乱来
}
递归版2，超简洁递归法!思路其实也是左插右前边
void flatten(TreeNode *root) {
    flattenCore(root, NULL);
}
TreeNode *flattenCore(TreeNode *root, TreeNode *tail) {//把 root 所代表树变成链表后，tail 跟在该链表后面
    if(root == NULL) return tail; //若root没有，返回的就是root后边的那个tail
    TreeNode *rightRoot = flattenCore(root->right, tail);   //整体root的tail就是right的tail
    TreeNode *leftRoot = flattenCore(root->left, rightRoot); //右边的root就是左边的tail，要连起来
    root->right = leftRoot;
    root->left = NULL;  //封口
    return root;    //返回的是root!
}
迭代版，感觉更好理解，推荐方法! 时间复杂度 O(n)，空间复杂度 O(logn)
void flatten(TreeNode *root) {
    //迭代，用栈!! 前序遍历，非常方便的方法，应该效率较高也
    if(root == NULL)    return;
    stack<TreeNode *> s;
    s.push(root);
    while(!s.empty()) {
        TreeNode *temp = s.top();
        s.pop();
        if(temp->right)  s.push(temp->right);
        if(temp->left)   s.push(temp->left);
        if(!s.empty())
            temp->right = s.top(); //串接到下一个要访问的元素
        temp->left = NULL;  //仍然要封口!
    }
}

21,  Binary Tree Maximum Path Sum  略难
Given a binary tree, find the maximum path sum.
The path may start and end at any node in the tree.
For example:
Given the below binary tree,
       1
      / \
     2   3
Return 6.
int maxPathSum(TreeNode *root) {
    //高级DFS，不一定都是正数，难!
    int max_sum = INT_MIN;
    dfs(root, max_sum);
    return max_sum;
}
int dfs(TreeNode *root, int &max_sum) {  //返回单个方向路径的最大值，max_sum一定是引用!
    if(root == NULL)
        return 0;
    int lLen = dfs(root->left, max_sum);
    int rLen = dfs(root->right, max_sum);
    int sum = (lLen>0?lLen:0) + (rLen>0?rLen:0) + root->val;    //只有大于，才加!!! 注意优先级!!!
    max_sum = max(max_sum, sum);
    return max(lLen, rLen) > 0 ? max(lLen, rLen) + root->val : root->val;   //若大于，才加进去
}

23, Subtree - 非Leetcode
You have two very large binary trees: T1, with millions of nodes, and T2 with hundreds of nodes.
Create an algorithm to decide if T2 is a subtree of T1. 
bool subTree(TreeNode* t1, TreeNode* t2) { 
    if (!t2) return true; 
    if (!t1) return false; 
    if(t1->val == t2->val) 
        if(matchTree(t1,t2)) return true; 
    return subTree(t1->left,t2) || subtree(t1->right,t2); 
} 
bool matchTree(TreeNode* t1, TreeNode* t2) { 
    if(!t1 && !t2) return true; 
    if(!t1 || !t2) return false; 
    if(t1->val != t2->val) return false; 
    return matchTree(t1->left,t2->left) && matchTree(t1->right,t2->right); 
}

24, Path Sum
Given a binary tree and a sum, determine if the tree has a root-to-leaf path such that adding up all the values along the path equals the given sum.
For example:
Given the below binary tree and sum = 22,
              5
             / \
            4   8
           /   / \
          11  13  4
         /  \      \
        7    2      1
return true, as there exist a root-to-leaf path 5->4->11->2 which sum is 22.
bool hasPathSum(TreeNode *root, int sum) {
    if(root == NULL)    return false;   //这样，只有一边有叶子的话，也包括进来了!
    if(root->left==NULL && root->right==NULL) //到了叶子
        return (sum == root->val);
    //if(root->left!=NULL && root->right!=NULL)  这样也包括进来了
    return hasPathSum(root->left, sum-root->val) || hasPathSum(root->right, sum-root->val);
}

25, Path Sum II
Given a binary tree and a sum, find all root-to-leaf paths where each path's sum equals the given sum.
For example:
Given the below binary tree and sum = 22,
              5
             / \
            4   8
           /   / \
          11  13  4
         /  \    / \
        7    2  5   1
return
[
   [5,4,11,2],
   [5,8,4,5]
]
vector<vector<int> > pathSum(TreeNode *root, int sum) {
    //求所有的，所以找到之后，不能返回，接着找 
    vector<vector<int> > res;
    vector<int> line;
    pathSumCore(root, sum, res, line);
    return res;
}
void pathSumCore(TreeNode *root, int sum, vector<vector<int> > &res, vector<int> &line) {
    if(root == NULL)
        return;
    line.push_back(root->val);
    if(root->left==NULL && root->right==NULL) {
        if(sum == root->val)
            res.push_back(line);
        line.pop_back();    //弹出来
        return;
    }
    pathSumCore(root->left, sum - root->val, res, line);  //接着找
    pathSumCore(root->right, sum - root->val, res, line);
    line.pop_back();
}
方法2，非递归：- 难，不考虑
//当年写剑指offer25题的时候，我自己发明的非递归，后序遍历解法!!!
vector<vector<int> > pathSum(TreeNode *root, int sum) {
    vector<vector<int> > res;
    std::vector<int> path;
    stack<TreeNode*> s;
    int currentSum=0;
    TreeNode* p=root, *q=root;    //定义2个指针，p周游，q记录已经访问过的节点
    while(p){
        for(;p->left!=NULL; p=p->left) {    //一直找到，最下边的左节点
            s.push(p);
            path.push_back(p->val);
            currentSum += p->val;
        }
        while(p!=NULL && (p->right==NULL || p->right==q)) {  //当前节点不空，没有右子树或右子树根节点已经访问过
            currentSum += p->val;  //后序遍历中的，访问节点
            path.push_back(p->val);
            if(p->right==NULL && p->left==NULL) {//叶节点
                if(currentSum == sum)     //若注释掉，打印的就是所有的根到叶节点的路径!!!
                    res.push_back(path);
            }
            currentSum -= path.back();        //该节点，算完sum，不保留，弹出去
            path.pop_back();
 
            q = p;        //记录上一次访问过的节点
            if(s.empty())       //结束!!!，总体来说，有2个叶节点的父节点都被压入了2次
                return res;
            //若栈空，结束，否则取出栈顶节点作为当前节点
            p = s.top();
            s.pop();
            //pop后，需要弹path
            currentSum -= path.back();
            path.pop_back();
        }
        s.push(p);  //将当前节点压入栈中
        //push后，相应压path
        path.push_back(p->val);
        currentSum += p->val;
        p = p->right;      //将当前节点的右子树的根节点设为当前节点，一直走到最下边
    }
    return res;
}

拓展：若不需要root起点，且不需要leaf终点：CC150 4.9
递归，对于每个访问到的点node，回溯搜一遍node到root的路径，加里边的和，看是否有以node结尾的这样的path

26, Balanced Binary Tree
Given a binary tree, determine if it is height-balanced.
For this problem, a height-balanced binary tree is defined as a binary tree in which the depth of the two subtrees of every node never differ by more than 1.
直观的方法：
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
    if( leftD-rightD > 1 || leftD-rightD < -1 ) //算术运算符>关系运算符>逻辑运算符
        return false;
    return isBalanced(root->left) && isBalanced(root->right);
}
新方法：如果按照上一种做法，直接对每个结点求depth再比较深度差，会有很多重复计算，某个节点会被遍历多次，求多次高度
进一步简洁，推荐：
bool isBalanced(TreeNode *root) {
    return balancedHeight (root) != -1;     //为了省去 higt的参数
}
int balancedHeight (TreeNode* root) {   //返回值：-1代表不balanced，其他返回高度
    if (root == NULL) return 0; // 终止条件 模板第1部分
    int left = balancedHeight (root->left);    // 处理left和right  模板第2部分
    int right = balancedHeight (root->right);
    if (left == -1 || right == -1 || abs(left - right) > 1)     // 处理自己的 模板第3部分
        return -1; // 剪枝
    return max(left, right) + 1; // 三方合并
}

28, Populating Next Right Pointers in Each Node
 Given a binary tree
    struct TreeLinkNode {
      TreeLinkNode *left;
      TreeLinkNode *right;
      TreeLinkNode *next;
    }
Populate each next pointer to point to its next right node. If there is no next right node, the next pointer should be set to NULL.
Initially, all next pointers are set to NULL.
Note:
You may only use constant extra space.
You may assume that it is a perfect binary tree (ie, all leaves are at the same level, and every parent has two children).
自己写的迭代算法，牛！
void connect(TreeLinkNode *root) {
    //O(1) 空间，从上一层得到下一层
    if(root == NULL)    return;
    TreeLinkNode *node=root;
    while(node->left != NULL) { //处理到叶子节点就返回
        TreeLinkNode *link = new TreeLinkNode(-1);  //伪节点，是为了实现link->next = p->left; 要不还得特殊判断这一行
        for(TreeLinkNode *p=node; p!=NULL; p=p->next) {
            link->next = p->left;
            link->next->next = p->right;
            link = link->next->next;
        }
        node = node->left;  //下一行开始
    }
}
DFS 递归方法，思路简洁!!
void connect(TreeLinkNode *root) {
    if(root == NULL)
        return;
    connectCore(root->left, root->right);
}
void connectCore(TreeLinkNode *leftSibling, TreeLinkNode *rightSibling) {
    if(leftSibling == NULL) //相当于判断rightSibling了也
        return;
    leftSibling->next = rightSibling;
    if(leftSibling->left == NULL)   //同样判断一个即可
        return;
    connectCore(leftSibling->left, leftSibling->right);
    connectCore(rightSibling->left, rightSibling->right);
    connectCore(leftSibling->right, rightSibling->left);    //二者之间
}

29, Populating Next Right Pointers in Each Node II
Follow up for problem "Populating Next Right Pointers in Each Node".
What if the given tree could be any binary tree? Would your previous solution still work?
自己写的迭代算法，仍然可以用，只是需要记录下linkhead+内部判断p->left！ 略难
void connect(TreeLinkNode *root) {
    //O(1) 空间，从上一层得到下一层
    if(root == NULL)    return;
    TreeLinkNode *head = root;  //伪节点，每一层开头
    while(head != NULL) { //处理到叶子节点就返回
        TreeLinkNode *linkhead = new TreeLinkNode(-1);  //下一层开头伪节点，是为了串起下边一排
        TreeLinkNode *link = linkhead;
        for(TreeLinkNode *p=head; p!=NULL; p=p->next) {
            if(p->left) {
                link->next = p->left;
                link = link->next;
            }
            if(p->right) {
                link->next = p->right;
                link = link->next;
            }
        }
        head = linkhead->next;  //head下一行开始
    }
}
递归方法，其实思路和上边一样：
void connect(TreeLinkNode *root) {
    if(root == NULL)    return;
    TreeLinkNode dummy(-1); //每一层的，伪头结点
    for(TreeLinkNode *curr=root, *prev=&dummy; curr; curr=curr->next) { //每一层，从头到尾进行链接
        if(curr->left) {
            prev->next = curr->left;
            prev = prev->next;
        }
        if(curr->right) {
            prev->next = curr->right;
            prev = prev->next;
        }
    }
    connect(dummy.next);    //每一层的开头的节点
}

32, Sum Root to Leaf Numbers
Given a binary tree containing digits from 0-9 only, each root-to-leaf path could represent a number.
An example is the root-to-leaf path 1->2->3 which represents the number 123.
Find the total sum of all root-to-leaf numbers.
For example,
    1
   / \
  2   3
The root-to-leaf path 1->2 represents the number 12.
The root-to-leaf path 1->3 represents the number 13.
Return the sum = 12 + 13 = 25.
int sumNumbers(TreeNode *root) {
    //DFS即可，可以统计
    int res=0, path=0;
    if(root == NULL)  return 0;
    sumNumbersCore(root, path, res);
    return res;
}
void sumNumbersCore(TreeNode *root, int path, int &res) { //path是中间结果，res是最终结果且需要&
    path = 10 * path + root->val;
    if(root->left==NULL && root->right==NULL) { //叶子节点
        res += path;
        return;
    }   //若不是叶子节点
    if(root->left)
        sumNumbersCore(root->left, path, res);
    if(root->right)
        sumNumbersCore(root->right, path, res);    
}
