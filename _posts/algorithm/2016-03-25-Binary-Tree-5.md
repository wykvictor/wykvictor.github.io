---
layout: post
title:  "Binary Tree 5 - Sum in Tree"
date:   2016-03-25 20:30:00
tags: [algorithm, leetcode, binary tree, sum]
categories: Algorithm
---

### 1. Two sum in a BST - Hard
```
Given a BST and a value x. Find two nodes in the tree whose sum is equal x.
Additional space: O(height of the tree). It is not allowed to modify the tree. 

The Brute Force Solution is to consider each pair in BST and check whether the sum equals to X. 
The time complexity of this solution will be O(n^2).

A Better Solution is to create an auxiliary array and store Inorder traversal of BST in the array.
The array will be sorted as Inorder traversal of BST always produces sorted data.
Once we have the Inorder traversal, we can pair in O(n) time.
This solution works in O(n) time, but requires O(n) auxiliary space.

A space optimized solution is to first in-place convert BST to Doubly Linked List (DLL),
then find pair in sorted DLL in O(n) time. 
This solution takes O(n) time and O(Logn) extra space, but it modifies the given BST.

The solution discussed below takes O(n) time, O(Logn) space and doesn’t modify BST.
The idea is same as finding the pair in sorted array.
We traverse BST in Normal Inorder and Reverse Inorder simultaneously.
In reverse inorder, we start from the rightmost node which is the maximum value node.
In normal inorder, we start from the left most node which is minimum value node.
We add sum of current nodes in both traversals and compare this sum with given target sum.
If the sum is same as target sum, we return true.
If the sum is more than target sum, we move to next node in reverse inorder traversal,
otherwise we move to next node in normal inorder traversal.
If any of the traversals is finished without finding a pair, we return false.
```

Following is C++ implementation of the last approach:
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
            if (curr1 != NULL){  // 中序遍历迭代版本
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

### 2. [Binary Tree Maximum Path Sum II](http://www.lintcode.com/en/problem/binary-tree-maximum-path-sum-ii/)
```
Given a binary tree, find the maximum path sum from root.
The path may end at any node in the tree and contain at least one node in it.
```
{% highlight C++ %}
int maxPathSum2(TreeNode *root) {
  if(root == NULL) {
    return 0;
  }
  int left = maxPathSum2(root->left);
  int right = maxPathSum2(root->right);
  if(left <= 0 && right <= 0) {
    return root->val;
  }
  // either of the 2 <isindex></isindex> > 0
  return root->val + (left > right ? left : right);
}
{% endhighlight %}

[Binary Tree Maximum Path Sum - Leetcode 124 Hard](https://leetcode.com/problems/binary-tree-maximum-path-sum/)

```
Given a binary tree, find the maximum path sum.
The path may start and end at any node in the tree.
For example:
Given the below binary tree,
       1
      / \
     2   3
Return 6.
```

// Use resultType, 推荐方法
{% highlight C++ %}
// need to know: root为根，答案是多少；到root最长多少
struct ResultType {
  int max_sum; // 从树中任意到任意点的最大路径，这条路径至少包含一个点
  int single_path; //从root往下走到任意点的最大路径,至少包含一个点root
  ResultType(int a, int b): max_sum(a), single_path(b) {}
};
ResultType maxPathSumHelper(TreeNode *root) {
  if(root == NULL) {
      return ResultType(INT_MIN, INT_MIN);
  }
  ResultType left = maxPathSumHelper(root->left);
  ResultType right = maxPathSumHelper(root->right);
  
  int single_path = root->val + 
    max(0, max(left.single_path, right.single_path));
  int max1 = max(left.max_sum, right.max_sum);
  int max2 = root->val +
    (left.single_path>0?left.single_path:0) +
    (right.single_path>0?right.single_path:0);
  int max_sum = max(max1, max2);
  return ResultType(max_sum, single_path);
}
int maxPathSum(TreeNode *root) {
  return maxPathSumHelper(root).max_sum;
}
{% endhighlight %}
取巧的方法，将ResultType拆到了函数参数上
{% highlight C++ %}
int maxPathSum(TreeNode *root) {
  //高级DFS，node存的不一定都是正数
  int max_sum = INT_MIN;
  dfs(root, max_sum);
  return max_sum;
}
//函数返回单个方向路径的最大值，max_sum则是引用 返回了整体的值!
int dfs(TreeNode *root, int &max_sum) {
  if(root == NULL)
      return 0;
  int lLen = dfs(root->left, max_sum);
  int rLen = dfs(root->right, max_sum);
  // 只有大于，才加!!! 注意优先级!!!
  int sum = (lLen>0?lLen:0) + (rLen>0?rLen:0) + root->val;
  max_sum = max(max_sum, sum);
  // 若大于，才加进去
  return max(lLen, rLen) > 0 ? max(lLen, rLen) + root->val : root->val;
}
{% endhighlight %}

### 3. [Path Sum - Leetcode 112](https://leetcode.com/problems/path-sum/)
```
Given a binary tree and a sum, determine if the tree has a root-to-leaf path
such that adding up all the values along the path equals the given sum.

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
```
{% highlight C++ %}
bool hasPathSum(TreeNode *root, int sum) {
  if(root == NULL)    return false;
  if(root->left==NULL && root->right==NULL) //到了叶子
      return (sum == root->val);
  //if(root->left!=NULL && root->right!=NULL) 开头那句包括了这种情况
  return hasPathSum(root->left, sum-root->val) || hasPathSum(root->right, sum-root->val);
}
{% endhighlight %}

### 4. [Path Sum II - Leetcode 113](https://leetcode.com/problems/path-sum-ii/)
```
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
```
{% highlight C++ %}
vector<vector<int> > pathSum(TreeNode *root, int sum) {
    //求所有的，所以找到之后，不能返回，接着找 
    vector<vector<int> > res;
    vector<int> line;
    pathSumCore(root, sum, res, line);
    return res;
}
void pathSumCore(TreeNode *root, int sum, vector<vector<int> > &res, vector<int> &line) {
    if(root == NULL)  // 必须判断；否则调用时需判断root->left!=NULL
        return;
    line.push_back(root->val);  // !先压再判断
    if(root->left==NULL && root->right==NULL) {
        if(sum == root->val)
            res.push_back(line);
        line.pop_back();    // 需要弹出来!
        return;
    }
    pathSumCore(root->left, sum - root->val, res, line);  //接着找
    pathSumCore(root->right, sum - root->val, res, line);
    line.pop_back();
}
{% endhighlight %}
方法2，非递归：- hard，不考虑此方法
{% highlight C++ %}
//当年写剑指offer25题的时候，自己发明的非递归，后序遍历解法
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
        // 当前节点不空，没有右子树或右子树根节点已经访问过
        while(p!=NULL && (p->right==NULL || p->right==q)) {
            currentSum += p->val;  // 后序遍历中的，访问节点
            path.push_back(p->val);
            if(p->right==NULL && p->left==NULL) {//叶节点
                if(currentSum == sum)     //若注释掉，打印的就是所有的根到叶节点的路径!
                    res.push_back(path);
            }
            currentSum -= path.back();        //该节点，算完sum，不保留，弹出去
            path.pop_back();
 
            q = p;        //记录上一次访问过的节点
            if(s.empty())     //结束!总体来说，有2个叶节点的父节点都被压入了2次
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
        p = p->right; //将当前节点的右子树的根节点设为当前节点，一直走到最下边
    }
    return res;
}
{% endhighlight %}
拓展：若不需要root起点，且不需要leaf终点：CC150 4.9

递归解法，对于每个访问到的点node，回溯搜一遍node到root的路径，加里边的和，看是否有以node结尾的这样的path

### 5. [Sum Root to Leaf Numbers - Leetcode 129](https://leetcode.com/problems/sum-root-to-leaf-numbers/)
```
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
```
{% highlight C++ %}
int sumNumbers(TreeNode *root) {
    //DFS即可，可以统计
    int res=0, path=0;
    if(root == NULL)  return 0;
    sumNumbersCore(root, path, res);
    return res;
}
//path是中间结果，res是最终结果且需要&
void sumNumbersCore(TreeNode *root, int path, int &res) {
    path = 10 * path + root->val;
    if(root->left==NULL && root->right==NULL) { //叶子节点
        res += path;
        return;
    }   //若不是叶子节点
    if(root->left)  sumNumbersCore(root->left, path, res);
    if(root->right)  sumNumbersCore(root->right, path, res);    
}
{% endhighlight %}

### 6. [Minimum Subtree](http://www.lintcode.com/en/problem/minimum-subtree/)
```
Given a binary tree, find the subtree with minimum sum. Return the root of the subtree.
```
{% highlight C++ %}
int findCore(TreeNode* root, TreeNode*& result, int& minSum) {
  if (root == NULL) return 0;
  // Divide & Conqur, bottom-up
  int left = findCore(root->left, result, minSum);
  int right = findCore(root->right, result, minSum);
  int curSum = left + right + root->val;
  // minSum Global, compare with it
  if (curSum < minSum) {
    minSum = curSum;
    result = root;
  }
  return curSum;
}
TreeNode* findSubtree(TreeNode* root) {
  TreeNode* result = NULL;
  int minSum = INT_MAX;
  findCore(root, result, minSum);  // 不可直接传INT_MAX!! 引用需是变量
  return result;
}
{% endhighlight %}

### 7. [Subtree with Maximum Average](http://www.lintcode.com/en/problem/subtree-with-maximum-average/)
```
Given a binary tree, find the subtree with maximum average. Return the root of the subtree.
```
{% highlight C++ %}
// 也可以把num放到返回值中：Result(sum;num), 代码结构更清晰!
int findCore(TreeNode* root, TreeNode*& result, double& maxAvg, int& num) {
  if (root == NULL) {
    num = 0;
    return 0;
  }
  // Divide & Conqur, bottom-up
  int leftNum = 0, rightNum = 0;
  int left = findCore(root->left, result, maxAvg, leftNum);
  int right = findCore(root->right, result, maxAvg, rightNum);
  int curSum = left + right + root->val;
  num = leftNum + rightNum + 1;  // 当前的节点数目
  // maxAvg Global, compare with it
  if ((double)curSum / num > maxAvg) {  // 注意：double!
    maxAvg = (double)curSum / num;
    result = root;
  }
  return curSum;
}

TreeNode* findSubtree2(TreeNode* root) {
  TreeNode* result = NULL;
  double maxAvg = INT_MIN;  // 注意：double!
  int num = 0;
  findCore(root, result, maxAvg, num);  // 不可直接传INT_MAX!! 引用需是变量
  return result;
}
{% endhighlight %}
