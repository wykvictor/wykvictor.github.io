---
layout: post
title:  "Array & Numbers 1 - Two Pointers"
date:   2016-03-27 10:30:00
tags: [algorithm, leetcode, array, two pointer]
categories: Algorithm
---

> 多指针/滑动窗口: 利用多个指针对一个数组或链表进行扫描，比如以不同速度从头向尾扫描，或分别从头和尾向中间扫描等等

> 使用多指针的目的 是尽量减少循环pass的次数以提高效率

> 需要注意的是扫描终止条件的判断

### 1. [Summary Ranges - Leetcode 228](https://leetcode.com/problems/summary-ranges/)
```
Given a sorted integer array without duplicates, return the summary of its ranges.
For example, given [0,1,2,4,5,7], return ["0->2","4->5","7"].
```

{% highlight C++ %}
vector<string> summaryRanges(vector<int>& nums) {
    vector<string> res;
    if(nums.size() == 0)  return res;
    int begin=0, end=0;  // the index
    while(end < nums.size()) {
        for(end=begin+1; end<nums.size(); end++) {
            if(nums[end] > nums[end-1] + 1) //越界!nums[end]-nums[end-1]>1
                break;
        }
        string line(to_string(nums[begin])); //另 boost::lexical_cast<string>()
        if(end > begin+1) {
            line += "->" + to_string(nums[end-1]);
        }
        res.push_back(line);
        begin = end;
    }
    return res;
}
{% endhighlight %}
 
### 2. [Remove duplicates from sorted array - Leetcode 26](https://leetcode.com/problems/remove-duplicates-from-sorted-array/)
Given a sorted array, remove the duplicates in place such that each element appear only once and return the new length.
Do not allocate extra space for another array, you must do this in place with constant memory.
For example,
Given input array A = [1,1,2],
Your function should return length = 2, and A is now [1,2].
int removeDuplicates(int A[], int n) {
    //2个指针，从前往后
    if(n == 0)  return 0;
    int i=0, j=1;
    for(; j<n; j++) {
        if(A[j] != A[i]) {
            A[++i] = A[j];  //其实不需要这个判断，if(j-i > 1) 另，i+1替换为i++;
        }
    }
    return i+1;
}

2，[Remove duplicates from sorted array II - Leetcode 80](https://leetcode.com/problems/remove-duplicates-from-sorted-array-ii/)
What if duplicates are allowed at most twice?
For example,
Given sorted array A = [1,1,1,2,2,3],
Your function should return length = 5, and A is now [1,1,2,2,3].
int removeDuplicates(int A[], int n) {
    if(n < 3)  return n;    //这里不同
    int i=1, j=2;       //初始化值不同
    for(; j<n; j++) {
        if(A[j] != A[i-1]) {    //这个变成了i-1，不是很好理解
            A[++i] = A[j];  //其实不需要这个判断，if(j-i > 1) 另，i+1替换为i++;
        } 
    }
    return i+1;
}
其实下边这样写最好：
int removeDuplicates(int A[], int n) {
    if(n<3)     return n;   //1或2个元素，直接返回
    //另一种，简单方法!!
    int index=2;
    for(int i=2; i<n; i++) {
        if(A[i] == A[index-1] && A[i] == A[index-2])    //只有这种情况，跳过，啥不干
            continue;
        A[index++] = A[i];
    }
    return index;
}

九章课的好懂的算法：
//用课上的，int dup实现下
int removeDuplicates(int A[], int n) {
    if(n<3)     return n;   //1或2个元素，直接返回
    int dup = 1;      //代表初始化，dup只有1个 ==> 该方法，可以扩展到多个dup的时候
    int index = 0;  //也是从0开始
    for(int i=1; i<n; i++) {
        if(A[i] != A[index]) {
            A[++index] = A[i];
            dup = 1;
        } else {
            if(dup < 2) {  //只有小于2的时候，才复制
                A[++index] = A[i];
                dup++;
            }
        }
    }
    return index+1; //注意，返回的是个数+1
}

3，[Longest substring without repeating characters - Leetcode 3](https://leetcode.com/problems/longest-substring-without-repeating-characters/)
Given a string, find the length of the longest substring without repeating characters. For example, the longest substring without repeating letters for "abcabcbb" is "abc", which the length is 3. For "bbbbb" the longest substring is "b", with the length of 1.
常规思路：2
int lengthOfLongestSubstring(string s) {
    //用bool数组记录是否出现过，因为一出现重复就需要重新计算i之前的数组，所以复杂度高
    bool hash[256] = {false};  //for(int k=0; k<256; k++)   hash[k] = false; 貌似不需要!
    int maxLen = 0;
    int i=0, j=0;   //一前，一后指针
    for(; j<s.size(); j++){
        if(hash[s[j]]) {
            maxLen = max(maxLen, j-i);
            while(s[i] != s[j]) {   //efabcabc 第1个a之前清除标志
                hash[s[i]] = false;
                i++;
            }
            i++;
        }
        hash[s[j]] = true;      //不管如何，都需要记录下标；或放到else里，上一种情况，记录了其实
    }
    return max(maxLen, j-i);    //必须最后再比较一下，防止最后一次的遗漏!
}
优化：用int hash记录位置，这样不需要回头搞，记录一个上次start开始的位置就可以  3
int lengthOfLongestSubstring(string s) {
    //更新后，时间复杂度 O(n)，空间复杂度 O(1)，因为i不往回倒退了，只走一遍!!
    int hash[256];   //为了记录位置，所以用int
    for(int i=0; i<256; i++)
        hash[i] = -1;
    int maxLen = 0;
    int i=0, j=0;   //一前，一后指针
    for(; j<s.size(); j++){
        if(hash[s[j]] >= i) {  //从开始点i开始算才有效
            maxLen = max(maxLen, (j-i));
            i = hash[s[j]] + 1; //新的开始点
        }
        hash[s[j]] = j; //不管是否命中，都需要记录最新的下标
    }
    return max(maxLen, j-i);  //必须最后再比较一下，防止最后一次的遗漏!
}

4，[Container With Most Water - Leetcode 11](https://leetcode.com/problems/container-with-most-water/)
Given n non-negative integers a1, a2, ..., an, where each represents a point at coordinate (i, ai). n vertical lines are drawn such that the two endpoints of line i is at (i, ai) and (i, 0). Find two lines, which together with x-axis forms a container, such that the container contains the most water.
Note: You may not slant the container.
自己想到的 1
int maxArea(vector<int> &height) {
    //两个指针，扫。注意，一前一后扫
    int start=0, end=height.size()-1;
    int maxW=0;
    while(start < end) {
        if(height[end] >= height[start]) {  //以start起始的，已经找到了
            maxW = max(maxW, height[start] * (end-start));
            start++;
            end = height.size()-1;
        } else {
            maxW = max(maxW, height[end] * (end-start));
            end--;
        }
    }
    return maxW;
}
优化：end不需要归位了，且抽取area计算到if else的外边  2
int maxArea(vector<int> &height) {
    //两个指针，扫。注意，以前以后扫
    int start=0, end=height.size()-1;
    int maxW=0;
    while(start < end) {
        maxW = max(maxW, min(height[start], height[end]) * (end-start));
        if(height[end] >= height[start]) {  //以start起始的，已经找到了
            start++;    //end = height.size()-1;  end不需要归位，end之后的东西都是短于前一个start的，没必要计算了
        } else {
            end--;
        }
    }
    return maxW;
}

12, [Trapping Rain Water - Leetcode 42](https://leetcode.com/problems/trapping-rain-water/)
Given n non-negative integers representing an elevation map where the width of each bar is 1, compute how much water it is able to trap after raining.
For example, 
Given [0,1,0,2,1,0,1,3,2,1,2,1], return 6.

The above elevation map is represented by array [0,1,0,2,1,0,1,3,2,1,2,1]. In this case, 6 units of rain water (blue section) are being trapped. 
扫两边，常规解法 1
int trap(int A[], int n) {
    //每根柱子分割开来，找到左、右边最高的，就能知道它上部能放多少水
    vector<int> left(n, 0);
    vector<int> right(n, 0);
    int high=0, res=0;
    for(int i=1; i<n; i++) {  //扫左边的  
        high = high > A[i-1] ? high : A[i-1];
        left[i] = high;
    }
    high=0;
    for(int i=n-2; i>=0; i--) {  //扫右边的 + 顺便计算
        high = high > A[i+1] ? high : A[i+1];
        right[i] = high;
        int temp = min(left[i], right[i]) - A[i];   //注意，不能加负的!
        if(temp > 0)    res += temp;
    }
    return res;
}
方法2，找中间最高 2
int trap(int A[], int n) {
    //扫描一遍，找到最高的柱子，这个柱子将数组分为两半；再分别处理左右2边
    int maxIndex=0;
    for(int i=0; i<n; i++)
        if(A[i] > A[maxIndex])  maxIndex = i;
    int res=0;
    for(int i=0, peak=0; i<maxIndex; i++) { //peak是左边最高点
        if(peak > A[i])    res += peak- A[i];
        else    peak = A[i];    //左边最高点，没自己高，不可能存水
    }
    for(int i=n-1, peak=0; i>maxIndex; i--) { //peak是左边最高点
        if(peak > A[i])    res += peak- A[i];
        else    peak = A[i];
    }
    return res;
}

5，[Two Sum - Leetcode 1](https://leetcode.com/problems/two-sum/)
Given an array of integers, find two numbers such that they add up to a specific target number.
The function twoSum should return indices of the two numbers such that they add up to the target, where index1 must be less than index2. Please note that your returned answers (both index1 and index2) are not zero-based.
You may assume that each input would have exactly one solution.
Input: numbers={2, 7, 11, 15}, target=9
Output: index1=1, index2=2
常规解法 1
bool mycompare(const pair<int, int> &num1, const pair<int, int> &num2) { //注意，这么写! 而且该函数，要写到class外边!!!
    return num1.first < num2.first; //不需要考虑相等的情况，因为each input would have exactly one solution
}
vector<int> twoSum(vector<int> &numbers, int target) {
    //先排序，然后左右夹逼，排序 O(n log n)，左右夹逼 O(n)，最终 O(n log n)。但是注意，这题需要返回的是下标,需pair
    vector<int> res;
    vector<pair<int, int> > numIndex;
    for(int i=0; i<numbers.size(); i++)
        numIndex.push_back(make_pair(numbers[i], i+1));
    sort(numIndex.begin(), numIndex.end(), mycompare);
     
    int i=0, j=numbers.size()-1;
    while(i < j) {
        if(numIndex[i].first + numIndex[j].first < target) {
            i++;
        } else if(numIndex[i].first + numIndex[j].first > target) {
            j--;
        } else {
            res.push_back(min(numIndex[i].second, numIndex[j].second));
            res.push_back(max(numIndex[i].second, numIndex[j].second));
            break;
        }
    }
    return res;
}
优化数据结构解法 2
vector<int> twoSum(vector<int> &numbers, int target) {
    //用一个哈希表hash，存储每个数对应的下标，复杂度 O(n)
    vector<int> res;
    unordered_map<int, int> mapping;
    for (int i = 0; i < numbers.size(); i++) {
        mapping[numbers[i]] = i+1;      //可以这么赋值
    }
    for (int i = 0; i < numbers.size(); i++) {
        const int gap = target - numbers[i];
        if (mapping.find(gap) != mapping.end() && mapping[gap] != (i+1)) {   //主要是这个查找，时间近似为O(1)
            res.push_back(i + 1);
            res.push_back(mapping[gap]); //gap对应的下标
            break;
        }
    }
    return res;
}

6，[3 Sum - Leetcode 15](https://leetcode.com/problems/3sum/)
Given an array S of n integers, are there elements a, b, c in S such that a + b + c = 0? Find all unique triplets in the array which gives the sum of zero.
Note:
Elements in a triplet (a,b,c) must be in non-descending order. (ie, a ≤ b ≤ c)
The solution set must not contain duplicate triplets.
    For example, given array S = {-1 0 1 2 -1 -4},
    A solution set is:
    (-1, 0, 1)
    (-1, -1, 2)
常规解法 2 先排序，再左右夹逼，将n^2降为nlogn
//这个方法可以推广到k-sum，先排序，然后做k-2次循环，在最内层循环左右夹逼，时间复杂度是O(max{nlogn,n^k-1})
vector<vector<int> > threeSum(vector<int> &num) {
    //先排序nlogn，再左右夹逼,两层循环n^2
    vector<vector<int> > res;
    if(num.size() < 3)
        return res;
    sort(num.begin(), num.end());   //注意，该函数若num为空，会崩溃!!!
    for(int i=0; i<num.size()-2 && num[i]<=0; i++) {   //最外层，从左到右; 而且一旦i大于0了，没必要再找了直接退出
        if(i!=0 && num[i]==num[i-1])    //防止重复
            continue;
        int j=i+1, k=num.size()-1;
        int gap = 0 - num[i];
        while(j < k) {
            if(num[j] + num[k] == gap)
                res.push_back({num[i], num[j], num[k]});    //这么初始化，简洁!!!
            if(num[j] + num[k] >= gap) {    //等于的时候，也需要移动j或k!!!
                do{
                    k--;
                }while(k>j && num[k]==num[k+1]);
            } else {
                do{
                    j++;
                }while(k>j && num[j]==num[j-1]);
            }
        }
    }
    return res;
}
超时的，简洁的解法
vector<vector<int> > threeSum(vector<int> &num) {
    //先排序nlogn，再左右夹逼,两层循环n^2 中途不判重的话，最后统一去也行
    vector<vector<int> > res;
    if(num.size() < 3)
        return res;
    sort(num.begin(), num.end());   //注意，该函数若num为空，会崩溃!!!
    for(int i=0; i<num.size()-2 && num[i]<=0; i++) {   //最外层，从左到右; 而且一旦i大于0了，没必要再找了直接退出
        int j=i+1, k=num.size()-1;
        int gap = 0 - num[i];
        while(j < k) {
            if(num[j] + num[k] == gap) {
                res.push_back({num[i], num[j], num[k]});    //这么初始化，简洁!!!
                k--;    j++;
            } else if(num[j] + num[k] > gap) {    //等于的时候，也需要移动j或k!!!
                k--;
            } else {
                j++;
            }
        }
    }
    sort(res.begin(), res.end()); 
    res.erase(unique(res.begin(), res.end()), res.end());   //unique返回排除重复后的结尾，erase删除以后的
    return res;
}
7，[3 Sum closest - Leetcode 16](https://leetcode.com/problems/3sum-closest/)
Given an array S of n integers, find three integers in S such that the sum is closest to a given number, target. Return the sum of the three integers. You may assume that each input would have exactly one solution.
For example, given array S = {-1 2 1 -4}, and target = 1.
The sum that is closest to the target is 2. (-1 + 2 + 1 = 2).
常规解法 1
int threeSumClosest(vector<int> &num, int target) {
    //思路一样 题目说有且只有1个答案，所以边界不用判断
    sort(num.begin(), num.end());
    int closest = INT_MAX, res = INT_MIN;
    for(int i=0; i<num.size()-2; i++) {
        int j=i+1, k=num.size()-1;
        while(j < k) {
            int s = num[i] + num[j] + num[k];
            if(s < target)
                j++;
            else if(s>target)
                k--;
            else
                return s;
            if(abs(s-target) < closest) {
                closest = abs(s-target);    //abs的使用!
                res = s;
            }
        }
    }
    return res;
}
8，[4 Sum - Leetcode 18](https://leetcode.com/problems/4sum/)
Given an array S of n integers, are there elements a, b, c, and d in S such that a + b + c + d = target? Find all unique quadruplets in the array which gives the sum of target.
Note:
Elements in a quadruplet (a,b,c,d) must be in non-descending order. (ie, a ≤ b ≤ c ≤ d)
The solution set must not contain duplicate quadruplets.
For example, given array S = {1 0 -1 0 -2 2}, and target = 0.
    A solution set is:
    (-1,  0, 0, 1)
    (-2, -1, 1, 2)
    (-2,  0, 0, 2)
常规解法 1
vector<vector<int> > fourSum(vector<int> &num, int target) {
    //和3sum很类似啊 n^3
    vector<vector<int> > res;
    if(num.size() < 4)
        return res;
    sort(num.begin(), num.end());
    for(int i=0; i<num.size()-3; i++) { //就是多加了一层循环
        if(i!=0 && num[i]==num[i-1])
            continue;
        for(int j=i+1; j<num.size()-2; j++) {
            if(j!=i+1 && num[j]==num[j-1])
                continue;
            int beg=j+1, end=num.size()-1;
            int gap = target - num[i] - num[j];
            while(beg < end) {
                if(num[beg]+num[end] == gap) {
                    res.push_back({num[i], num[j], num[beg], num[end]});
                    do {
                        beg++;
                    } while(beg < end && num[beg] == num[beg-1]);   //一直移动到beg不等，end不用管了其实
                } else if(num[beg]+num[end] < gap) {
                    beg++;  //如果下一个beg相同，还是进来，所以没必要里边循环也
                } else {
                    end--;
                }
            }
        }
    }
    return res;
}
同样的，简洁的解法，最后统一去重 2
vector<vector<int> > fourSum(vector<int> &num, int target) {
    //最后统一去重
    vector<vector<int> > res;
    if(num.size() < 4)  return res;
    sort(num.begin(), num.end());
    for(int i=0; i<num.size()-3; i++) {
        for(int j=i+1; j<num.size()-2; j++) {
            int begin=j+1, end=num.size()-1;
            while(begin < end) {
                int sum = num[i] + num[j] + num[begin] + num[end];
                if(sum < target)
                    begin++;
                else if(sum > target)
                    end--;
                else {
                    res.push_back({num[i], num[j], num[begin], num[end]});
                    begin++;    //别忘了!!
                    end--;
                }
            }
        }
    }
    sort(res.begin(), res.end());   //注意也要这样!! “删除”所有 相邻 的重复元素
    res.erase(unique(res.begin(), res.end()), res.end());
    return res;
}
牛一点的解法，不太建议用，也是用了hash：
vector<vector<int> > fourSum(vector<int> &num, int target) {
    //高级方法 用一个 hashmap 先缓存两个数的和
    //时间复杂度，平均 O(n^2)，最坏 O(n^4)，空间复杂度 O(n^2)
    vector<vector<int> > res;
    int size = num.size();
    if(size < 4)  return res;
    sort(num.begin(), num.end());
     
    unordered_map<int, vector<pair<int, int> > > cache; //vector内存所有的和是int的pair
    for (int a = 0; a < size-1; ++a) {
        for (int b = a + 1; b < size; ++b) {
            cache[num[a] + num[b]].push_back(pair<int, int>(a, b)); //有重复的等于一个值的:存下标!方便下边过滤重复
        }
    }
/*
hashTable[2sum] = {(num1, num2), (num3, num4)}
(1,2,3,4)

target: 8
(1,2), (2,3)
 3, 5
O(N^2)
*/
    for (int c = 0; c < size-3; ++c) {
        for (int d = c + 1; d < size-2; ++d) {
            const int key = target - num[c] - num[d];
            if (cache.find(key) == cache.end()) continue;//没有找到
            vector<pair<int, int> > vec = cache[key];
            for (int k = 0; k < vec.size(); ++k) {
                if (vec[k].first <= d) //若c是第一个数，d是第2个,则pair中的第一个数的下标必须大于d
                    continue; // 有重叠 第
                res.push_back({num[c], num[d], num[vec[k].first], num[vec[k].second]});
            }   
        }
    }
    sort(res.begin(), res.end());   //注意也要这样!! “删除”所有 相邻 的重复元素
    res.erase(unique(res.begin(), res.end()), res.end());
    return res;
}
9，[Merge sorted array - Leetcode 88](https://leetcode.com/problems/merge-sorted-array/) 
Given two sorted integer arrays A and B, merge B into A as one sorted array.
Note:
You may assume that A has enough space (size that is greater or equal to m + n) to hold additional elements from B. 
The number of elements initialized in A and B are m and n respectively.
简洁解法!
void merge(int A[], int m, int B[], int n) {
    //分别一个指针，扫. 为了省空间，从后往前赋值 时间复杂度 O(m+n)，空间复杂度 O(1)
    int i=m-1, j=n-1, index=m+n-1;  //需要有个index指针
    while(i>=0 && j>=0) {
        A[index--] = A[i] >= B[j] ? A[i--] : B[j--];
    }
    while(j>=0) {     //不需要再弄i的了
        A[index--] = B[j--];
    }
}
10, [Implement strStr - Leetcode 28](https://leetcode.com/problems/implement-strstr/)
Returns a pointer to the first occurrence of needle in haystack, or null if needle is not part of haystack.
常规解法 算法课Java版本:

C++解法 2
char *strStr(char *haystack, char *needle) {
    //暴力解法，时间复杂度 O(N*M)，空间复杂度 O(1),大集合超时
    //优化：实际上只需要循环n-m+1次，因为长度不够m的时候肯定不可能匹配
    //设个指针tail，开始时tail与开始间隔m，当tail指到字符串结束出，循环停止!
    if(needle == NULL || *needle == '\0')
        return haystack;        //特殊情况，可以问面试官!  
    char *p=haystack, *q=needle, *pp=haystack;
    char *tail = haystack;
    while(*++q != '\0') //这个++q或q++可以自己模拟算下
        tail++;
    for(; *tail!='\0'; p++, tail++) {
        for(pp = p, q=needle; *pp!='\0' && *q!='\0'; pp++, q++) {  //每次q归位!
            if(*pp != *q)  //只有相等，才加，不能用pp++ == q++这样的!!!
                break;
        }
        if(*q == '\0')
            return p;
    }
    return NULL; //到结尾
}
C++解法 简化求长度
char *strStr(char *haystack, char *needle) {
    //暴力解法，时间复杂度 O(N*M)，空间复杂度 O(1),大集合超时
    //优化：实际上只需要循环n-m+1次，因为长度不够m的时候肯定不可能匹配
    //设个指针tail，开始时tail与开始间隔m，当tail指到字符串结束出，循环停止!  或者用strlen
    if(needle == NULL || *needle == '\0')
        return haystack;        //特殊情况，可以问面试官!  
    char *p=haystack, *q=needle, *pp=haystack;
    int lenN = strlen(needle);
    for(; *(p+lenN-1)!='\0'; p++) {
        for(pp = p, q=needle; *pp!='\0' && *q!='\0'; pp++, q++) {  //每次q归位!
            if(*pp != *q)  //只有相等，才加，不能用pp++ == q++这样的!!!
                break;
        }
        if(*q == '\0')
            return p;
    }
    return NULL; //到结尾
}
11, [Substring with Concatenation of All Words - Leetcode 30](https://leetcode.com/problems/substring-with-concatenation-of-all-words/)
You are given a string, S, and a list of words, L, that are all of the same length. Find all starting indices of substring(s) in S that is a concatenation of each word in L exactly once and without any intervening characters.
For example, given:
S: "barfoothefoobarman"
L: ["foo", "bar"]
You should return the indices: [0,9].
(order does not matter)
这个主要用map这样的数据结构 略难
vector<int> findSubstring(string S, vector<string> &L) {
    //主要考察数据结构的使用
    int wLen = L.front().size();    //用front!
    int tLen = wLen * L.size();
    vector<int> res;
    if(tLen > S.size()) return res;
     
    unordered_map<string, int> times;
    for(int i=0; i<L.size(); i++) {
        times[L[i]]++;
    }
    for(int i=0; i<S.size() - tLen + 1; i++) {  //注意这种找字符串的题，一定优化到不够长!注意+1
        map<string, int> has_found;     //初始化全是0
        int j=0;
        for(; j < L.size(); j++) {  //依次扫描L中的每一个单词，但不一定按L中的顺序!
            string t = S.substr(i + j * wLen, wLen);    //每次跳过S中的一个单词
            if(times.find(t) == times.end())  //没找到时, 这2个时要跳出
                break;
            ++has_found[t];
            if(has_found[t] > times[t])  //数量不对时
                break;
        }
        if(j == L.size()) {
            res.push_back(i);
        }
    }
    return res;
}

13, [Valid palindrome - Leetcode 125](https://leetcode.com/problems/valid-palindrome/)
Given a string, determine if it is a palindrome, considering only alphanumeric characters and ignoring cases.
For example,
"A man, a plan, a canal: Panama" is a palindrome.
"race a car" is not a palindrome.
Note:
Have you consider that the string might be empty? This is a good question to ask during an interview.
For the purpose of this problem, we define empty string as valid palindrome.
常规解法，代码已极度精简
bool isPalindrome(string s) {
    //正常逻辑处理，先过滤字符，后判断
    string ss;
    for(int i=0; i<s.size(); i++) {
        if( (s[i]>='0' && s[i]<='9') || (s[i]>='a' && s[i]<='z') || (s[i]>='A' && s[i]<='Z') )
            ss += (s[i]>='A' && s[i]<='Z') ? (s[i] + 'a' - 'A'): s[i]; //ss[ssindex++]错!应该用+=动态增长
    }
    int i=0, j=ss.size()-1;
    while(i < j) { //不用判断==的情况
        if(ss[i++] != ss[j--])  //比下边调用i++; j--;更简洁
            return false;
    }
    return true;    //we define empty string as valid palindrome
}
14, [Reverse Integer - Leetcode 7](https://leetcode.com/problems/reverse-integer/)
Reverse digits of an integer.
Example1: x = 123, return 321
Example2: x = -123, return -321
click to show spoilers.
Have you thought about this?
Here are some good questions to ask before coding. Bonus points for you if you have already thought through this!
If the integer's last digit is 0, what should the output be? ie, cases such as 10, 100.
Did you notice that the reversed integer might overflow? Assume the input is a 32-bit integer, then the reverse of 1000000003 overflows. How should you handle such cases?
Throw an exception? Good, but what if throwing an exception is not an option? You would then have to re-design the function (ie, add an extra parameter).
没考虑溢出的，极度精简的代码：
int reverse(int x) {
    //时间复杂度 O(lgn)，空间复杂度 O(1),短小代码!
    int res=0;
    for(; x; x /= 10) { //一般for循环，都比while简洁
        res = 10*res + x % 10;  //若x%10开头为0，结果是对的
    }
    return res; //负数，也是对的，不需要单独处理.  这里只是要提一下，溢出怎么办?
}
溢出，这个更建议在面试的时候写：
int reverse(int x) {
    //考虑溢出
    double res=0;   //这样的话，不会溢出
    for(; x; x /= 10) { //一般for循环，都比while简洁
        res = 10*res + x % 10;  //若x%10开头为0，结果是对的
    }
    if(res < INT_MIN)   return INT_MIN; //不需要if(neg)这样，都判断一下不就行了，2！
    if(res > INT_MAX)   return INT_MAX;
    return (int)res;
}
15, [Palindrome Number - Leetcode 9](https://leetcode.com/problems/palindrome-number/)
Determine whether an integer is a palindrome. Do this without extra space.
click to show spoilers.
Some hints:
Could negative integers be palindromes? (ie, -1)
If you are thinking of converting the integer to string, note the restriction of using extra space.
You could also try reversing an integer. However, if you have solved the problem "Reverse Integer", you know that the reversed integer might overflow. How would you handle such case?
There is a more generic way of solving this problem.
若直接用reverse，会溢出，所以下面直接首尾比较，靠谱
bool isPalindrome(int x) {
    //参考reverse的方法，直接一个一个比呗
    if(x < 0)   return false;    //若负数，也算Palindrome的话==>问面试官
    int base=1;
    while(x / base >= 10)    //find base，注意有==!
        base *= 10;
    for(; x>0; base /= 100) {    //一次比较首尾位2位，所以除以100；另，x是个位数就跳出是true(错，10021)
        if(x % 10 != x / base)
            return false;
        x = x % base / 10;      //x -= x / base * base;
    }
    return true;
}
16, [Minimum window substring - Leetcode 76 Hard](https://leetcode.com/problems/minimum-window-substring/)
Given a string S and a string T, find the minimum window in S which will contain all the characters in T in complexity O(n).
For example,
S = "ADOBECODEBANC"
T = "ABC"
Minimum window is "BANC".
Note:
If there is no such window in S that covers all characters in T, return the emtpy string "".
If there are multiple such windows, you are guaranteed that there will always be only one unique minimum window in S.
双指针的使用，right一直往前，left在count到的情况下收缩
string minWindow(string S, string T) {
    //用hash记录T，然后扫S，2个指针典型的题
    int lenT = T.size();
    string res = "";  //返回的是""
    int needFind[256] = {0};    //有可能出现多次，所以用int
    int hasFound[256] = {0};
    for(int i=0; i<lenT; i++)
       needFind[T[i]]++;
     
    int count=0, minLen = INT_MAX;
    for(int left=0, right=0; right<S.size(); right++) {
        if(needFind[S[right]] == 0)    //只关心相关的字母
            continue;
        hasFound[S[right]]++;   //每次碰到都加，但是count不用加
        if(hasFound[S[right]] <= needFind[S[right]])    //注意，有等于！
            count++;
        if(count == lenT) { //说明找全乎了，可以收缩left了
            while(left <= right && (!needFind[S[left]] || hasFound[S[left]]>needFind[S[left]])) {
                if(hasFound[S[left]]>needFind[S[left]])
                    hasFound[S[left]]--;
                left++;
            }
            if(right-left+1 < minLen) {
                minLen = right-left+1;
                res = S.substr(left, minLen);
            }
        }
    }
    return res;
}
17, [Sort colors - Leetcode 75](https://leetcode.com/problems/sort-colors/)
Given an array with n objects colored red, white or blue, sort them so that objects of the same color are adjacent, with the colors in the order red, white and blue.
Here, we will use the integers 0, 1, and 2 to represent the color red, white, and blue respectively.
Note:
You are not suppose to use the library's sort function for this problem.
Follow up:
A rather straight forward solution is a two-pass algorithm using counting sort.
First, iterate the array counting number of 0's, 1's, and 2's, then overwrite array with total number of 0's, then 1's and followed by 2's.
Could you come up with an one-pass algorithm using only constant space?
void sortColors(int A[], int n) {
    //Follow up中的方案:排序的范围就3个数，用桶排序呗==>O(n)时间，O(1)空间
    int timesofA[3]={0};    //初始化!! 否则Runtime Error
    for(int i=0; i<n; i++)
        timesofA[A[i]]++;
    int index=0;
    for(int i=0; i<3; i++)
        while(timesofA[i]-- > 0)
            A[index++] = i;
}

void sortColors(int A[], int n) {
    //更好的方法，一遍遍历就解决，3个指针
    int last=n-1, first=0; //反正就3种数字,前后交换呗,两边往中间走
    int index=0;    //遍历的下标
    while(index <= last) {
        if(A[index] == 0) {
            swap(A[first++], A[index++]);   //0的时候，index可以加,因为换出来的最大是1
        }else if(A[index] == 2) {
            swap(A[index], A[last--]);  //!!注意，这种情况不能index++!!因为不知道换回来的是啥
        } else {
            index++;
        }
    }        
}

void sortColors(int A[], int n) {
    //利用快速排序里partition的思想，第一次将数组按0分割，第二次按1分割，排序完毕
    //可以推广到 n种颜色，每种颜色有重复元素的情况。
    int index = partition(A, 0, n-1, 0);
    partition(A, index, n-1, 1);    
}
int partition(int A[], int start, int end, int pivot) { //稍微变形，传入pivot为1或者0
    int low=start, high=end;
    while(low <= high) {
        while(low <= high && A[low] <= pivot)
            low++;
        while(low <= high && A[high] > pivot)
            high--;
        if(low < high) {
            swap(A[high], A[low]);
            low++;  high--;
        }
    }
    return low; //low肯定是对的
}
18, [First Missing Positive - Leetcode 41](https://leetcode.com/problems/first-missing-positive/)
Given an unsorted integer array, find the first missing positive integer.
For example,
Given [1,2,0] return 3,
and [3,4,-1,1] return 2.
Your algorithm should run in O(n) time and uses constant space.
int firstMissingPositive(int A[], int n) {
    //用标记数组bool，出现的true，第一个没出现的就是答案，但是空间O(n)，不靠谱!
    //有个方法，各就各位:每当 A[i]!= i+1 的时候，将 A[i] 与 A[A[i]-1] 交换，直到无法交换为止
    for(int i=0; i<n; i++) { //先进行 各就各位 操作
        while(A[i] != i+1 && A[i]>0 && A[i]<=n && A[i] != A[A[i] - 1]) {   //0<A[i]<=n, 且防止无休止swap
            swap(A[i], A[A[i]-1]);
        }
    }
    for(int i=0; i<n; i++) {    //找结果
        if(A[i] != i+1)
            return i+1;
    }
    return n+1;
}
另，类似的题Find missing number，已经排序了，用二分
You have an array a[] and the length n, the array should filled from 0 to n-1 but now one 
number missed. Find the missing number. 
For example, to the array {0,1,3,4,5,6,7}, the missing number is 2. 
int findMissing(int a[], int n){ 
  int left = 0; 
  int right = n-1; 
  while (left <= right){ 
    int m = (left+right) / 2; 
    if(m!=0 && a[m-1]+1!=a[m]) 
      return a[m]-1; 
    if(m==0 && a[m]!=0)  //注意missing number如果是
第0个情况的考虑
      return 0; 
    if(m!=n && a[m+1]-1 != a[m]) 
      return a[m]+1; 
  
    if (a[m] == m) 
      left = m+1; 
    else 
      right = m-1; 
  } 
  return -1; 
} 

19, [Remove Element - Leetcode 27](https://leetcode.com/problems/remove-element/)
Given an array and a value, remove all instances of that value in place and return the new length.
The order of elements can be changed. It doesn't matter what you leave beyond the new length. 
自己的笨办法：类似快排partition，其实这样拷贝的次数少，时间比较优化，但是代码冗余
int removeElement(int A[], int n, int elem) {
    int left=0, right=n-1;
    while(left <= right) {
        while(left <= right && A[left] != elem)
            left++;
        while(left <= right && A[right] == elem)
            right--;
        if(left > right)    return left;
        swap(A[left], A[right]);
        left++;
        right--;
    }
    return left;
}
快慢指针法：
int removeElement(int A[], int n, int elem) {
    int low=0, high=0;
    for(; high<n; high++) {
        if(A[high] != elem) //相等的时候，high往前走，不用复制
            A[low++] = A[high];
    }
    return low;
}
