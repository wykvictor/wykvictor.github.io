---
layout: post
title:  "Array & Numbers 4 - Others"
date:   2016-03-30 10:30:00
tags: [algorithm, leetcode, array, two pointer]
categories: Algorithm
---

### 1. [Longest substring without repeating characters - Leetcode 3](https://leetcode.com/problems/longest-substring-without-repeating-characters/)
```
Given a string, find the length of the longest substring without repeating characters.
For example, the longest substring without repeating letters for "abcabcbb" is "abc", which the length is 3.
For "bbbbb" the longest substring is "b", with the length of 1.
```

常规思路:
{% highlight C++ %}
int lengthOfLongestSubstring(string s) {
  //用bool数组记录是否出现过，因为一出现重复就需要重新计算i之前的数组，所以复杂度略高
  bool hash[256] = {false};  // 统一初始化C数组
  int maxLen = 0;
  int i = 0, j = 0;  // 一前，一后指针
  for (; j < s.size(); j++) {
    if (hash[s[j]]) {
      maxLen = max(maxLen, j - i);
      while (s[i] != s[j]) {  // efabcabc 第1个a之前清除标志
        hash[s[i]] = false;
        i++;
      }
      i++;
    }
    //不管如何，都需要记录下标；或放到else里，上一种情况，记录了其实
    hash[s[j]] = true;
  }
  return max(maxLen, j-i); //必须最后再比较一下，防止最后一次的遗漏!
}
{% endhighlight %}
优化：用int hash记录位置，这样不需要回头搞，记录一个上次start开始的位置就可以
{% highlight C++ %}
int lengthOfLongestSubstring(string s) {
  //更新后，时间复杂度 O(n)，空间复杂度 O(1)，因为i不往回倒退了，只走一遍!!
  int hash[256];  //为了记录位置，所以用int
  for (int i = 0; i < 256; i++) hash[i] = -1;
  int maxLen = 0;
  int i = 0, j = 0;  //一前，一后指针
  for (; j < s.size(); j++) {
    if (hash[s[j]] >= i) {  //!从开始点i开始算才有效
      maxLen = max(maxLen, (j - i));
      i = hash[s[j]] + 1;  //新的开始点
    }
    hash[s[j]] = j; //不管是否命中，都需要记录最新的下标
  }
  return max(maxLen, j-i);  //必须最后再比较一下，防止最后一次的遗漏!
}
{% endhighlight %}

### 2. [Container With Most Water - Leetcode 11](https://leetcode.com/problems/container-with-most-water/)
```
Given n non-negative integers a1, a2, ..., an, where each represents a point at coordinate (i, ai).
n vertical lines are drawn such that the two endpoints of line i is at (i, ai) and (i, 0).
Find two lines, which together with x-axis forms a container, such that the container contains the most water.
```

自己想到的:
{% highlight C++ %}
int maxArea(vector<int> &height) {
  //两个指针，扫。注意，一前一后扫
  int start = 0, end = height.size() - 1;
  int maxW = 0;
  while (start < end) {
    if (height[end] >= height[start]) {  //以start起始的，已经找到了
      maxW = max(maxW, height[start] * (end - start));
      start++;
      end = height.size() - 1;  // end归到最后，重新算
    } else {
      maxW = max(maxW, height[end] * (end - start));
      end--;
    }
  }
  return maxW;
}
{% endhighlight %}
优化：end不需要归位了，且抽取area计算到if else的外边
{% highlight C++ %}
int maxArea(vector<int> &height) {
  int start = 0, end = height.size() - 1;
  int maxW = 0;
  while (start < end) {
    maxW = max(maxW, min(height[start], height[end]) * (end - start));
    if (height[end] >= height[start]) {  //以start起始的，已经找到了
      start++;  // end = height.size()-1; // end不需要归位，end之后的东西都是短于前一个start的，没必要计算了
    } else {
      end--;
    }
  }
  return maxW;
}
{% endhighlight %}

### 3. [Trapping Rain Water - Leetcode 42](https://leetcode.com/problems/trapping-rain-water/)
```
Given n non-negative integers representing an elevation map where the width of each bar is 1, compute how much water it is able to trap after raining.
For example, 
Given [0,1,0,2,1,0,1,3,2,1,2,1], return 6.
```

扫两边，常规解法
{% highlight C++ %}
int trap(int A[], int n) {
  // 每根柱子分割开来，找到左、右边最高的，就能知道它上部能放多少水
  vector<int> left(n, 0);
  vector<int> right(n, 0);
  int high = 0, res = 0;
  for (int i = 1; i < n; i++) {  // 扫左边的
    high = high > A[i - 1] ? high : A[i - 1];
    left[i] = high;
  }
  high = 0;
  for (int i = n - 2; i >= 0; i--) {  // 扫右边的 + 顺便计算
    high = high > A[i + 1] ? high : A[i + 1];
    right[i] = high;
    int temp = min(left[i], right[i]) - A[i];
    if(temp > 0)    res += temp;    // 注意，不能加负的!
  }
  return res;
}
{% endhighlight %}
方法2，找中间最高
{% highlight C++ %}
int trap(int A[], int n) {
  //扫描一遍，找到最高的柱子，这个柱子将数组分为两半；再分别处理左右2边
  int maxIndex = 0;
  for (int i = 0; i < n; i++)
    if (A[i] > A[maxIndex]) maxIndex = i;
  int res = 0;
  for (int i = 0, peak = 0; i < maxIndex; i++) {  // peak是左边最高点
    if (peak > A[i])   res += peak - A[i];
    else    peak = A[i];  //左边最高点，没自己高，不可能存水
  }
  for (int i = n - 1, peak = 0; i > maxIndex; i--) {  // peak是左边最高点
    if(peak > A[i])    res += peak- A[i];
    else    peak = A[i];
  }
  return res;
}
{% endhighlight %}

### 4. [Substring with Concatenation of All Words - Leetcode 30](https://leetcode.com/problems/substring-with-concatenation-of-all-words/)
```
You are given a string, S, and a list of words, L, that are all of the same length. Find all starting indices of substring(s) in S that is a concatenation of each word in L exactly once and without any intervening characters.
For example, given:
S: "barfoothefoobarman"
L: ["foo", "bar"]
You should return the indices: [0,9].
(order does not matter)
```

这个主要用map这样的数据结构 略难
{% highlight C++ %}
vector<int> findSubstring(string S, vector<string> &L) {
  //主要考察数据结构的使用
  int wLen = L.front().size();  //用front!
  int tLen = wLen * L.size();
  vector<int> res;
  if (tLen > S.size()) return res;

  unordered_map<string, int> times;
  for (int i = 0; i < L.size(); i++) {
    times[L[i]]++;  // int不是bool，可能有重复的
  }
  // 注意这种找字符串的题，一定优化到不够长!注意+1
  for (int i = 0; i < S.size() - tLen + 1; i++) {
    map<string, int> has_found;  //初始化全是0
    int j = 0;
    for (; j < L.size(); j++) {  //依次扫描L中的每一个单词，但不一定按L中的顺序!
      string t = S.substr(i + j * wLen, wLen);  //每次跳过S中的一个单词
      if (times.find(t) == times.end())  //没找到时, 这2个时要跳出
        break;
      ++has_found[t];
      if (has_found[t] > times[t])  //数量不对时
        break;
    }
    if(j == L.size()) {
      res.push_back(i);
    }
  }
  return res;
}
{% endhighlight %}

### 5. [Valid palindrome - Leetcode 125](https://leetcode.com/problems/valid-palindrome/)
```
Given a string, determine if it is a palindrome, considering only alphanumeric characters and ignoring cases.
For example,
"A man, a plan, a canal: Panama" is a palindrome.
"race a car" is not a palindrome.
Note:
Have you consider that the string might be empty? This is a good question to ask during an interview.
For the purpose of this problem, we define empty string as valid palindrome.
```

常规解法，代码已极度精简
{% highlight C++ %}
bool isPalindrome(string s) {
  //正常逻辑处理，先过滤字符，后判断
  string ss;
  for (int i = 0; i < s.size(); i++) {
    if ((s[i] >= '0' && s[i] <= '9') || (s[i] >= 'a' && s[i] <= 'z') ||
        (s[i] >= 'A' && s[i] <= 'Z'))
      // ss[ssindex++]错!应该用+=动态增长
      ss += (s[i] >= 'A' && s[i] <= 'Z') ? (s[i] + 'a' - 'A') : s[i];
  }
  int i = 0, j = ss.size() - 1;
  while (i < j) {            //不用判断==的情况
    if (ss[i++] != ss[j--])  //比下边调用i++; j--;更简洁
      return false;
  }
  return true;  // we define empty string as valid palindrome
}
{% endhighlight %}

### 6. [Reverse Integer - Leetcode 7](https://leetcode.com/problems/reverse-integer/)
```
Reverse digits of an integer.
Example1: x = 123, return 321
Example2: x = -123, return -321
click to show spoilers.
Have you thought about this?
Here are some good questions to ask before coding. Bonus points for you if you have already thought through this!
If the integer's last digit is 0, what should the output be? ie, cases such as 10, 100.
Did you notice that the reversed integer might overflow? Assume the input is a 32-bit integer, then the reverse of 1000000003 overflows. How should you handle such cases?
Throw an exception? Good, but what if throwing an exception is not an option? You would then have to re-design the function (ie, add an extra parameter).
```

没考虑溢出的，极度精简的代码：
{% highlight C++ %}
int reverse(int x) {
  //时间复杂度 O(lgn)，空间复杂度 O(1),短小代码!
  int res = 0;
  for (; x; x /= 10) {        //一般for循环，都比while简洁
    res = 10 * res + x % 10;  //若x%10开头为0，结果是对的
  }
  return res; //负数，也是对的，不需要单独处理.  这里只是要提一下，溢出怎么办?
}
{% endhighlight %}
溢出，这个更建议在面试的时候写：
{% highlight C++ %}
int reverse(int x) {
  //考虑溢出
  long long res = 0;             //这样的话，不会溢出
  for (; x; x /= 10) {        //一般for循环，都比while简洁
    res = 10 * res + x % 10;  //若x%10开头为0，结果是对的
  }
  if (res < INT_MIN) return INT_MIN;
  if (res > INT_MAX) return INT_MAX;
  return (int)res;
}
{% endhighlight %}

### 7. [Palindrome Number - Leetcode 9](https://leetcode.com/problems/palindrome-number/)
Determine whether an integer is a palindrome. Do this without extra space.

Some hints:
Could negative integers be palindromes? (ie, -1)
If you are thinking of converting the integer to string, note the restriction of using extra space.
You could also try reversing an integer. However, if you have solved the problem "Reverse Integer", you know that the reversed integer might overflow. How would you handle such case?
There is a more generic way of solving this problem.
若直接用reverse，会溢出，所以下面直接首尾比较，靠谱
{% highlight C++ %}
bool isPalindrome(int x) {
  //参考reverse的方法，直接一个一个比呗
  if (x < 0) return false;  //若负数，也算Palindrome的话==>问面试官
  int base = 1;
  while (x / base >= 10)  // find base，注意有==!
    base *= 10;
  //一次比较首尾位2位，所以除以100；另，x是个位数就跳出是true(错，10021)
  for (; x > 0; base /= 100) {
    if (x % 10 != x / base) return false;
    x = x % base / 10;      //x -= x / base * base;
  }
  return true;
}
{% endhighlight %}

### 8. [Minimum window substring - Leetcode 76 Hard](https://leetcode.com/problems/minimum-window-substring/)
Given a string S and a string T, find the minimum window in S which will contain all the characters in T in complexity O(n).
For example,
S = "ADOBECODEBANC"
T = "ABC"
Minimum window is "BANC".
Note:
If there is no such window in S that covers all characters in T, return the emtpy string "".
If there are multiple such windows, you are guaranteed that there will always be only one unique minimum window in S.
双指针的使用，right一直往前，left在count到的情况下收缩
{% highlight C++ %}
string minWindow(string S, string T) {
  //用hash记录T，然后扫S，2个指针典型的题
  int lenT = T.size();
  string res = "";          //返回的是""
  int needFind[256] = {0};  //有可能出现多次，所以用int
  int hasFound[256] = {0};
  for (int i = 0; i < lenT; i++) needFind[T[i]]++;

  int count = 0, minLen = INT_MAX;
  for (int left = 0, right = 0; right < S.size(); right++) {
    if (needFind[S[right]] == 0)  //只关心相关的字母
      continue;
    hasFound[S[right]]++;  //每次碰到都加，但是count不用加
    if (hasFound[S[right]] <= needFind[S[right]])  //注意，有等于！
      count++;
    if (count == lenT) {  //说明找全乎了，可以收缩left了
      while (left <= right &&
             (!needFind[S[left]] || hasFound[S[left]] > needFind[S[left]])) {
        if (hasFound[S[left]] > needFind[S[left]]) hasFound[S[left]]--;
        left++;
      }
      if (right - left + 1 < minLen) {
        minLen = right-left+1;
        res = S.substr(left, minLen);
      }
    }
  }
  return res;
}
{% endhighlight %}

### 9. [First Missing Positive - Leetcode 41](https://leetcode.com/problems/first-missing-positive/)
Given an unsorted integer array, find the first missing positive integer.
For example,
Given [1,2,0] return 3,
and [3,4,-1,1] return 2.
Your algorithm should run in O(n) time and uses constant space.
{% highlight C++ %}
int firstMissingPositive(int A[], int n) {
  //用标记数组bool，出现的true，第一个没出现的就是答案，但是空间O(n)，不靠谱!
  //有个方法，各就各位:每当 A[i]!= i+1 的时候，将 A[i] 与 A[A[i]-1]
  //交换，直到无法交换为止
  for (int i = 0; i < n; i++) {  //先进行 各就各位 操作
    while (A[i] != i + 1 && A[i] > 0 && A[i] <= n &&
           A[i] != A[A[i] - 1]) {  // 0<A[i]<=n, 且防止无休止swap
      swap(A[i], A[A[i] - 1]);
    }
  }
  for(int i = 0; i < n; i++) {    //找结果
    if(A[i] != i+1)
      return i+1;
  }
  return n+1;
}
{% endhighlight %}

### 10. [Find missing number](http://www.lintcode.com/en/problem/find-the-missing-number/)
```
You have an array a[] and the length n, the array should filled from 0 to n-1 but now one 
number missed. Find the missing number. 
For example, to the array {0,1,3,4,5,6,7}, the missing number is 2. 
```

类似的题,已经排序了，用二分
{% highlight C++ %}
int findMissing(vector<int> &nums) {
  if (nums.size() == 0 || nums[0] != 0) return 0;  // 缺0单独考虑
  int start = 0, end = nums.size() - 1;
  while (start + 1 < end) {
    int mid = start + (end - start) / 2;
    if (nums[mid] == mid) {
      start = mid;
    } else {  // 缺的在前边
      end = mid;
    }
  }
  if (nums[end] == end) return end + 1;  // 如果end没错位，那肯定是end+1
  return start + 1;
}
{% endhighlight %}
如果是没排序的数组，跟上题相同
{% highlight C++ %}
int findMissing(vector<int> &nums) {
  int i = 0;
  for (; i < nums.size(); i++) {
    while (nums[i] != i && nums[i] < nums.size()) swap(nums[i], nums[nums[i]]);
  }  // 不存在重复的数字，所以不用判断nums[i]!=nums[nums[i]]
  for (int j = 0; j < nums.size(); j++)
    if (nums[j] != j) return j;
  return nums.size();
}
{% endhighlight %}

### 11. [Pow(x, n)](https://leetcode.com/problems/powx-n/#/description)
{% highlight C++ %}
double pow(double x, int n) {
  if (n < 0)
    return 1.0 / power(x, -n);
  else
    return power(x, n);
}
double power(double x, int n) {
  if (n == 0) return 1;
  double v = power(x, n / 2);
  if (n % 2 == 0)
    return v * v;
  else
    return v * v * x;
}
{% endhighlight %}
