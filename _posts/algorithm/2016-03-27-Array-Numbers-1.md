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

### 1. [Two Sum - Leetcode 1](https://leetcode.com/problems/two-sum/)
```
Given an array of integers, find two numbers such that they add up to a specific target number.
The function twoSum should return indices of the two numbers such that they add up to the target, where index1 must be less than index2. Please note that your returned answers (both index1 and index2) are not zero-based.
You may assume that each input would have exactly one solution.
Input: numbers={2, 7, 11, 15}, target=9
Output: index1=1, index2=2
```

常规解法, O(n) Space, O(nlogn) Time
{% highlight C++ %}
vector<int> twoSum(vector<int> &numbers, int target) {
  //先排序，然后左右夹逼，排序 O(n log n)，左右夹逼 O(n)，最终 O(nlogn)
  //但是注意，这题需要返回的是下标,需pair
  vector<int> res;
  vector<pair<int, int> > numIndex;  // 用map不行，有重复元素!
  for (int i = 0; i < numbers.size(); i++)
    numIndex.push_back(make_pair(numbers[i], i + 1));
  sort(numIndex.begin(), numIndex.end());  // 默认按first排序!

  int i = 0, j = numbers.size() - 1;
  while (i < j) {
    if (numIndex[i].first + numIndex[j].first < target) {
      i++;
    } else if (numIndex[i].first + numIndex[j].first > target) {
      j--;
    } else {
      res.push_back(min(numIndex[i].second, numIndex[j].second));
      res.push_back(max(numIndex[i].second, numIndex[j].second));
      break;
    }
  }
  return res;
}
{% endhighlight %}
优化数据结构解法，O(n) Space, O(n) Time
{% highlight C++ %}
vector<int> twoSum(vector<int> &numbers, int target) {
  vector<int> res;
  unordered_map<int, int> hash;  // 存储每个数对应的下标，复杂度 O(n)
  // 好! 一边循环每个数，一边加入hash表
  for (int i = 0; i < numbers.size(); i++) {
    // 主要是这个查找，时间近似为O(1)
    if (hash.find(target - numbers[i]) != hash.end()) {
      res.push_back(hash[target - numbers[i]]);  // gap对应的下标,这个更小
      res.push_back(i + 1);
      break;
    }
    hash[numbers[i]] = i + 1;  //可以这么赋值
  }
  return res;
}
{% endhighlight %}

### 2. [Two Sum - Data structure design](http://www.lintcode.com/en/problem/two-sum-data-structure-design/) 
{% highlight C++ %}
class TwoSum {
 public:
  unordered_multiset<int> nums;
  // Add the number to an internal data structure.
  void add(int number) { nums.insert(number); }
  // Find if there exists any pair of numbers which sum is equal to the value.
  bool find(int value) {
    for (auto i : nums) {
      int c = (i == value - i ? 2 : 1);  // 特殊情况，需要两个i
      if (nums.count(value - i) >= c) return true;
    }
    return false;
  }
};
{% endhighlight %}

### 3. [Two Sum - Unique pairs](http://www.lintcode.com/en/problem/two-sum-unique-pairs/)
```
Given an array of integers, find how many unique pairs in the array such that their sum is equal to a specific target number. Please return the number of pairs.
Given nums = [1,1,2,45,46,46], target = 47
return 2
1 + 46 = 47
2 + 45 = 47
```

Hash解法：易错
{% highlight C++ %}
int twoSum6(vector<int> &nums, int target) {
  unordered_map<int, int> um;
  for (auto i : nums) {
    if (um.find(i) != um.end())
      um[i]++;
    else
      um[i] = 1;
  }
  int res = 0;
  for (auto i : um) {
    if (um.find(target - i.first) == um.end()) continue;
    if (i.first == target - i.first) {  // 特殊情况，需要两个i
      if (um[target - i.first] >= 2) res += 2;
    } else {
      res += 1;
    }
  }
  return res / 2;  // 重复算了2遍
}
{% endhighlight %}
Two pointer解法：
{% highlight C++ %}
int twoSum6(vector<int> &nums, int target) {
  if (nums.size() <= 1) return 0;
  sort(nums.begin(), nums.end());
  int i = 0, j = nums.size() - 1;
  int res = 0;
  while (i < j) {
    if (nums[i] + nums[j] == target) {
      res++;
      do {
        i++;
      } while (i < j && nums[i] == nums[i - 1]);  // 别忘了i<j !!!
      do {
        j--;
      } while (i < j && nums[j] == nums[j + 1]);
    } else if (nums[i] + nums[j] < target) {
      i++;
    } else {
      j--;
    }
  }
  return res;
}
{% endhighlight %}

### 4. [Two Sum Closest](http://www.lintcode.com/en/problem/two-sum-closest/)
{% highlight C++ %}
// 返回最接近target的diff值是多少
int twoSumCloset(vector<int>& nums, int target) {
  sort(nums.begin(), nums.end());

  int closet = INT_MAX, i = 0, j = nums.size() - 1;
  while (i < j) {
    if (nums[i] + nums[j] < target) {
      closet = min(closet, target - nums[i] - nums[j]);
      i++;
    } else if (nums[i] + nums[j] > target) {
      closet = min(closet, nums[i] + nums[j] - target);
      j--;
    } else {
      return 0;
    }
  }
  return closet;
}
{% endhighlight %}

### 5. [3 Sum - Leetcode 15](http://www.lintcode.com/en/problem/3sum/)
```
Given an array S of n integers, are there elements a, b, c in S such that a + b + c = 0? Find all unique triplets in the array which gives the sum of zero.
Note:
Elements in a triplet (a,b,c) must be in non-descending order. (ie, a ≤ b ≤ c)
The solution set must not contain duplicate triplets.
    For example, given array S = {-1 0 1 2 -1 -4},
    A solution set is:
    (-1, 0, 1)
    (-1, -1, 2)
```

常规解法 先排序，再左右夹逼，Time是O(nlogn+n^2),Space是O(1)

优于hash解法，O(n^2)时间，O(n)空间
{% highlight C++ %}
//这个方法可以推广到k-sum，先排序，然后做k-2次循环，在最内层循环左右夹逼，时间复杂度是O(max{nlogn,n^k-1})
vector<vector<int> > threeSum(vector<int> &num) {
  //先排序nlogn，再左右夹逼,两层循环n^2
  vector<vector<int> > res;
  if (num.size() < 3) return res;
  sort(num.begin(), num.end());  // 注意，该函数若num为空，会崩溃!!!
  // 最外层，从左到右; 而且一旦i大于0了，没必要再找了直接退出，和不可能为0
  for (int i = 0; i < num.size() - 2 && num[i] <= 0; i++) {
    if (i != 0 && num[i] == num[i - 1])  // 优化：防止重复,i开头的已经算过了
      continue;
    int j = i + 1, k = num.size() - 1;
    int gap = 0 - num[i];
    while (j < k) {
      if (num[j] + num[k] == gap) {
        res.push_back({num[i], num[j], num[k]});  //这么初始化，简洁!!!
        j++;
        k--;
        while (k > j && num[j] == num[j - 1]) {  // 优化，防止重复
          j++;
        }
        while (k > j && num[k] == num[k + 1]) {  // 优化
          k--;
        }
      } else if (num[j] + num[k] > gap) {  //等于的时候，也需要移动j或k!!!
        k--;
      } else {
        j++;
      }
    }
  }
  return res;
}
{% endhighlight %}
可能超时的，简洁的解法：中途不判重的话，最后统一去也行
{% highlight C++ %}
vector<vector<int> > threeSum(vector<int> &num) {
  vector<vector<int> > res;
  if (num.size() < 3) return res;
  sort(num.begin(), num.end());  //注意，该函数若num为空，会崩溃!!!
  for (int i = 0; i < num.size() - 2 && num[i] <= 0; i++) {
    int j = i + 1, k = num.size() - 1;
    int gap = 0 - num[i];
    while (j < k) {
      if (num[j] + num[k] == gap) {
        res.push_back({num[i], num[j], num[k]});  //这么初始化，简洁!!!
        k--;
        j++;
      } else if (num[j] + num[k] > gap) {  //等于的时候，也需要移动j或k!!!
        k--;
      } else {
        j++;
      }
    }
  }
  sort(res.begin(), res.end());
  // unique返回排除重复后的结尾，erase删除以后的
  res.erase(unique(res.begin(), res.end()), res.end());
  return res;
}
{% endhighlight %}

### 6. [3 Sum closest - Leetcode 16](http://www.lintcode.com/en/problem/3sum-closest/)
```
Given an array S of n integers, find three integers in S such that the sum is closest to a given number, target. Return the sum of the three integers. You may assume that each input would have exactly one solution.
For example, given array S = {-1 2 1 -4}, and target = 1.
The sum that is closest to the target is 2. (-1 + 2 + 1 = 2).
```

常规解法
{% highlight C++ %}
int threeSumClosest(vector<int> num, int target) {
  sort(num.begin(), num.end());
  int closest = INT_MAX, res = INT_MIN;
  for (int i = 0; i < num.size() - 2; i++) {
    int j = i + 1, k = num.size() - 1;
    while (j < k) {
      int s = num[i] + num[j] + num[k];
      if (s < target)
        j++;
      else if (s > target)
        k--;
      else
        return s;
      if (abs(s - target) < closest) {
        closest = abs(s - target);
        res = s;
      }
    }
  }
  return res;
}
{% endhighlight %}

### 7. [4 Sum - Leetcode 18](https://leetcode.com/problems/4sum/)
```
Given an array S of n integers, are there elements a, b, c, and d in S such that a + b + c + d = target? Find all unique quadruplets in the array which gives the sum of target.
Note:
Elements in a quadruplet (a,b,c,d) must be in non-descending order. (ie, a ≤ b ≤ c ≤ d)
The solution set must not contain duplicate quadruplets.
For example, given array S = {1 0 -1 0 -2 2}, and target = 0.
    A solution set is:
    (-1,  0, 0, 1)
    (-2, -1, 1, 2)
    (-2,  0, 0, 2)
```

常规解法
{% highlight C++ %}
vector<vector<int> > fourSum(vector<int> &num, int target) {
  //和3sum很类似, n^3
  vector<vector<int> > res;
  if (num.size() < 4) return res;
  sort(num.begin(), num.end());
  for (int i = 0; i < num.size() - 3; i++) {  //就是多加了一层循环
    if (i != 0 && num[i] == num[i - 1]) continue;
    for (int j = i + 1; j < num.size() - 2; j++) {
      if (j != i + 1 && num[j] == num[j - 1]) continue;
      int beg = j + 1, end = num.size() - 1;
      int gap = target - num[i] - num[j];
      while (beg < end) {
        if (num[beg] + num[end] == gap) {
          res.push_back({num[i], num[j], num[beg], num[end]});
          beg++;
          end--;
          while (beg < end && num[beg] == num[beg - 1]) beg++;
          while (beg < end && num[end] == num[end + 1]) end--;
        } else if (num[beg] + num[end] < gap) {
          beg++;  //如果下一个beg相同，还是进来，所以没必要里边循环也
        } else {
          end--;
        }
      }
    }
  }
  return res;
}
{% endhighlight %}
牛一点的解法，不太建议用，也是用了hash：
{% highlight C++ %}
vector<vector<int> > fourSum(vector<int> &num, int target) {
  //高级方法 用一个 hashmap 先缓存两个数的和
  //时间复杂度，平均 O(n^2)，最坏 O(n^4)，空间复杂度 O(n^2)
  vector<vector<int> > res;
  int size = num.size();
  if (size < 4) return res;
  sort(num.begin(), num.end());

  // vector内存所有的和是int的pair
  unordered_map<int, vector<pair<int, int> > > cache;
  for (int a = 0; a < size - 1; ++a) {
    for (int b = a + 1; b < size; ++b) {
      //有重复的等于一个值的:存下标!方便下边过滤重复
      cache[num[a] + num[b]].push_back(pair<int, int>(a, b));
    }
  }
  for (int c = 0; c < size - 3; ++c) {
    for (int d = c + 1; d < size - 2; ++d) {
      const int key = target - num[c] - num[d];
      if (cache.find(key) == cache.end()) continue;  //没有找到
      vector<pair<int, int> > vec = cache[key];
      for (int k = 0; k < vec.size(); ++k) {
        //若c是第一个数，d是第2个,则pair中的第一个数的下标必须大于d
        if (vec[k].first <= d) continue;  // 有重叠，不要
        res.push_back({num[c], num[d], num[vec[k].first], num[vec[k].second]});
      }   
    }
  }
  sort(res.begin(), res.end());   //注意也要这样!! “删除”所有 相邻 的重复元素
  res.erase(unique(res.begin(), res.end()), res.end());
  return res;
}
{% endhighlight %}

### 8. [Partition Array](http://www.lintcode.com/en/problem/partition-array/)
```
Given an array nums of integers and an int k, partition the array such that:

All elements < k are moved to the left
All elements >= k are moved to the right
Return the partitioning index, i.e the first index i nums[i] >= k.

Can you partition the array in-place and in O(n)?
```
{% highlight C++ %}
int partitionArray(vector<int> &nums, int k) {
  int i = 0, j = nums.size() - 1;
  while (i <= j) {  // 比i<j好，不会死循环
    while (i <= j && nums[i] < k) i++;
    while (i <= j && nums[j] >= k) j--;
    if (i <= j) {
      swap(nums[i], nums[j]);
      i++;
      j--;
    }
  }
  return i;  // j+1
}
{% endhighlight %}

### 9. [Sort Letters by Case](http://www.lintcode.com/en/problem/sort-letters-by-case/)
```
Given a string which contains only letters. Sort it by lower case first and upper case second.
It's NOT necessary to keep the original order of lower-case letters and upper case letters.
Do it in one-pass and in-place.
```
{% highlight C++ %}
void sortLetters(string &letters) {
  int i = 0, j = letters.size() - 1;
  while (i <= j) {
    while (i <= j && letters[i] <= 'z' && letters[i] >= 'a') i++;
    while (i <= j && letters[j] <= 'Z' && letters[j] >= 'A') j--;
    if (i <= j) {
      swap(letters[i], letters[j]);
      i++;
      j--;
    }
  }
}
{% endhighlight %}

### 10. [Sort colors](http://www.lintcode.com/en/problem/sort-colors/)
```
Given an array with n objects colored red, white or blue, sort them so that objects of the same color are adjacent, with the colors in the order red, white and blue.
Here, we will use the integers 0, 1, and 2 to represent the color red, white, and blue respectively.
Note:
You are not suppose to use the library's sort function for this problem.
Follow up:
A rather straight forward solution is a two-pass algorithm using counting sort.
First, iterate the array counting number of 0's, 1's, and 2's, then overwrite array with total number of 0's, then 1's and followed by 2's.
Could you come up with an one-pass algorithm using only constant space?
```
{% highlight C++ %}
// counting sort
void sortColors(vector<int> &A) {
  // Follow up中的方案:排序的范围就3个数，用桶排序呗==>O(n)时间，O(1)空间
  int timesofA[3] = {0};  //初始化!! 否则Runtime Error
  for (int i = 0; i < A.size(); i++) timesofA[A[i]]++;
  int index = 0;
  for (int i = 0; i < 3; i++)
    while (timesofA[i]-- > 0) A[index++] = i;
}
{% endhighlight %}
重要！三分法：Two Pointers:
{% highlight C++ %}
void sortColors(vector<int> &A) {
  //更好的方法，一遍遍历就解决，3个指针
  int last = A.size() - 1, first = 0;  //反正就3种数字,前后交换呗,两边往中间走
  int index = 0;                       //遍历的下标
  while (index <= last) {  // !注意 <=, not <
    if (A[index] == 0) {
      swap(A[first++], A[index++]);  // 0的时候，index可以加,因为换出来的最大是1
    } else if (A[index] == 2) {
      //!!注意，这种情况不能index++!!因为不知道换回来的是啥
      swap(A[index], A[last--]);
    } else {
      index++;
    }
  }        
}
{% endhighlight %}
快速排序，2次partition思想:
{% highlight C++ %}
void sortColors(vector<int> &A) {
  //利用快速排序里partition的思想，第一次将数组按0分割，第二次按1分割，排序完毕
  //可以推广到 n种颜色，每种颜色有重复元素的情况。
  int index = partition(A, 0, 0);
  partition(A, index, 1);
}
int partition(vector<int> &A, int start, int pivot) {
  int low = start, high = A.size() - 1;
  while (low <= high) {
    while (low <= high && A[low] <= pivot) low++;
    while (low <= high && A[high] > pivot) high--;
    if (low < high) {
      swap(A[high], A[low]);
      low++;  high--;
    }
  }
  return low; // low肯定是对的
}
{% endhighlight %}

### 11. [Sort colors II](http://www.lintcode.com/en/problem/sort-colors-ii/)
```
n objects with k different colors
A rather straight forward solution is a two-pass algorithm using counting sort. That will cost O(k) extra memory. Can you do it without using extra memory?
```
{% highlight C++ %}
void sortColors2(vector<int> &colors, int k) {
  int start = 0, colorID = 1;
  while (start < colors.size() && colorID <= k) {
    start = partition(colors, start, colorID++);
  }
}
int partition(vector<int> &A, int start, int pivot) {
  int end = A.size() - 1;
  while (start <= end) {
    while (start <= end && A[start] <= pivot) start++;
    while (start <= end && A[end] > pivot) end--;
    if (start < end) {
      swap(A[end], A[start]);
      start++;
      end--;
    }
  }
}
{% endhighlight %}

### 12. [Interleaving Positive and Negative Numbers](http://www.lintcode.com/en/problem/interleaving-positive-and-negative-numbers/)
```
Given an array with positive and negative integers. Re-range it to interleaving with positive and negative integers.
Do it in-place and without extra memory
```
{% highlight C++ %}
void rerange(vector<int> &A) {
  int pos = 1, neg = 0;
  int posNum = 0, negNum = 0;
  for (auto a : A) {
    if (a > 0)
      posNum++;
    else
      negNum++;
  }
  if (posNum > negNum) {
    pos = 0;
    neg = 1;
  }
  while (pos < A.size() && neg < A.size()) {
    while (pos < A.size() && A[pos] > 0) pos += 2;
    while (neg < A.size() && A[neg] < 0) neg += 2;
    if (pos < A.size() && neg < A.size()) {
      swap(A[pos], A[neg]);
      pos += 2;
      neg += 2;
    }
  }
}
{% endhighlight %}

### 13. [Summary Ranges - Leetcode 228](https://leetcode.com/problems/summary-ranges/)
```
Given a sorted integer array without duplicates, return the summary of its ranges.
For example, given [0,1,2,4,5,7], return ["0->2","4->5","7"].
```
{% highlight C++ %}
vector<string> summaryRanges(vector<int>& nums) {
  vector<string> res;
  if (nums.size() == 0) return res;
  int begin = 0, end = 0;  // the index
  while (end < nums.size()) {
    for (end = begin + 1; end < nums.size(); end++) {
      if (nums[end] > nums[end - 1] + 1)  //越界!nums[end]-nums[end-1]>1
        break;
    }
    string line(to_string(nums[begin]));  //另 boost::lexical_cast<string>()
    if (end > begin + 1) {
      line += "->" + to_string(nums[end - 1]);
    }
    res.push_back(line);
    begin = end;
  }
  return res;
}
{% endhighlight %}
 
### 14. [Remove duplicates from sorted array - Leetcode 26](https://leetcode.com/problems/remove-duplicates-from-sorted-array/)
```
Given a sorted array, remove the duplicates in place such that each element
appear only once and return the new length.
Do not allocate extra space for another array, you must do this in place with constant memory.
For example,
Given input array A = [1,1,2],
Your function should return length = 2, and A is now [1,2].
```
{% highlight C++ %}
int removeDuplicates(int A[], int n) {
  // 2个指针，从前往后
  if (n == 0) return 0;
  int i = 0, j = 1;
  for (; j < n; j++) {
    if (A[j] != A[i])
      A[++i] = A[j];  //其实不需要这个判断，if(j-i > 1) 另，i+1替换为i++;
  }
  return i+1;
}
{% endhighlight %}

### 15. [Remove duplicates from sorted array II - Leetcode 80](https://leetcode.com/problems/remove-duplicates-from-sorted-array-ii/)
```
What if duplicates are allowed at most twice?
For example,
Given sorted array A = [1,1,1,2,2,3],
Your function should return length = 5, and A is now [1,1,2,2,3].
```
{% highlight C++ %}
int removeDuplicates(int A[], int n) {
  if (n < 3) return n;  //这里不同
  int i = 1, j = 2;     //初始化值不同
  for (; j < n; j++) {
    if (A[j] != A[i - 1])  //这个变成了i-1，不是很好理解
      A[++i] = A[j];  //其实不需要这个判断，if(j-i > 1) 另，i+1替换为i++;
  }
  return i+1;
}
{% endhighlight %}
其实下边这样写最好,好理解：
{% highlight C++ %}
int removeDuplicates(int A[], int n) {
  if (n < 3) return n;  // 1或2个元素，直接返回
  //另一种，简单方法!!
  int index = 2;
  for (int i = 2; i < n; i++) {
    //只有这种情况，跳过，啥不干
    if (A[i] == A[index - 1] && A[i] == A[index - 2]) continue;
    A[index++] = A[i];
  }
  return index;
}
{% endhighlight %}
课上int dup的算法：
{% highlight C++ %}
int removeDuplicates(int A[], int n) {
  if (n < 3) return n;  // 1或2个元素，直接返回
  int dup = 1;          // 代表初始化，dup只有1个 ==> 该方法，可以扩展到多个dup的时候
  int index = 0;        // 也是从0开始
  for (int i = 1; i < n; i++) {
    if (A[i] != A[index]) {
      A[++index] = A[i];
      dup = 1;
    } else {
      if (dup < 2) {  //只有小于2的时候，才复制
        A[++index] = A[i];
        dup++;  // 适用dup为3等等...
      }
    }
  }
  return index+1; //注意，返回的是个数+1
}
{% endhighlight %}

### 16. [Remove Element - Leetcode 27](https://leetcode.com/problems/remove-element/)
```
Given an array and a value, remove all instances of that value in place and return the new length.
The order of elements can be changed. It doesn't matter what you leave beyond the new length. 
```

类似快排partition，拷贝的次数少，时间比较优化，但是代码冗余
{% highlight C++ %}
int removeElement(int A[], int n, int elem) {
  int left = 0, right = n - 1;
  while (left <= right) {
    while (left <= right && A[left] != elem) left++;
    while (left <= right && A[right] == elem) right--;
    if (left > right) return left;
    swap(A[left], A[right]);
    left++;
    right--;
  }
  return left;
}
{% endhighlight %}
快慢指针法：
{% highlight C++ %}
int removeElement(int A[], int n, int elem) {
  int low = 0, high = 0;
  for (; high < n; high++) {
    if (A[high] != elem)  // 相等的时候，high往前走，不用复制
      A[low++] = A[high]; // copy较多
  }
  return low;
}
{% endhighlight %}

### 17. [Implement strStr - Leetcode 28](https://leetcode.com/problems/implement-strstr/)
{% highlight C++ %}
char *strStr(char *haystack, char *needle) {
  //暴力解法，时间复杂度 O(N*M)，空间复杂度 O(1),大集合超时
  //优化：实际上只需要循环n-m+1次，因为长度不够m的时候肯定不可能匹配
  //设个指针tail，开始时tail与开始间隔m，当tail指到字符串结束出，循环停止!
  //简化求长度: 用strlen
  if (needle == NULL || *needle == '\0')
    return haystack;  //特殊情况，可以问面试官!
  char *p = haystack, *q = needle, *pp = haystack;
  int lenN = strlen(needle);
  for (; *(p + lenN - 1) != '\0'; p++) {
    // 每次q归位!
    for (pp = p, q = needle; *pp != '\0' && *q != '\0'; pp++, q++) {
      if(*pp != *q)  //只有相等，才加，不能用pp++ == q++这样的!!!
        break;
    }
    if(*q == '\0')
      return p; // 找到啦
  }
  return NULL; //到结尾
}
// C++
int strStr(string &source, string &target) {
    // Write your code here
    // return source.find(target);
    int srclen = source.size();
    int tarlen = target.size();
    if (tarlen == 0)
        return 0;
    for(int i=0; i<srclen-tarlen+1; i++) { // srclen-tarlen+1
        int j=0;
        for(; j<tarlen; j++) {
            if(source[i+j] != target[j]) break;
        }
        if(j == tarlen) return i;
    }
    return -1;
}
{% endhighlight %}

### 18. [Wiggle Sort](http://www.lintcode.com/en/problem/wiggle-sort/)
```
Given an unsorted array nums, reorder it in-place such that
nums[0] <= nums[1] >= nums[2] <= nums[3]....
Please complete the problem in-place.
```
{% highlight C++ %}
// O(nlogn): First sorted, than flip each pair of 2 elements
void wiggleSort(vector<int>& nums) {
  sort(nums.begin(), nums.end());
  for (int i = 2; i < nums.size(); i += 2) swap(nums[i], nums[i - 1]);
}
// O(n), one pass
void wiggleSort(vector<int>& nums) {
  int size = nums.size();
  if (size < 2) return;
  for (int i = 1; i < nums.size(); i++) {
    if ((i % 2 == 1 && nums[i] < nums[i - 1]) ||
        (i % 2 == 0 && nums[i] > nums[i - 1]))
      swap(nums[i], nums[i - 1]);
  }
}
{% endhighlight %}

[Wiggle Sort II](http://www.lintcode.com/en/problem/wiggle-sort-ii/)
```
nums[0] < nums[1] > nums[2] < nums[3]....
```
{% highlight C++ %}
void wiggleSort(vector<int>& nums) {
  int size = nums.size();
  if (size < 2) return;
  vector<int> tmp(nums);
  sort(tmp.begin(), tmp.end());
  int mid = (size + 1) / 2;
  int j = mid - 1, k = size - 1; // 从后往前，选着放
  for (int i = 0; i < mid; i++) {
    nums[i * 2] = tmp[j--];
    if (i * 2 + 1 < size) nums[i * 2 + 1] = tmp[k--]; // 防止越界
  }
}
void wiggleSort(vector<int>& nums) {
#define A(i) nums[(1 + 2 * i) % (n | 1)]
  int n = nums.size(), i = 0, j = 0, k = n - 1;
  auto midptr = nums.begin() + n / 2;
  nth_element(nums.begin(), midptr, nums.end());
  int mid = *midptr;
  while (j <= k) {
    if (A(j) > mid)
      swap(A(i++), A(j++));
    else if (A(j) < mid)
      swap(A(j), A(k--));
    else
      ++j;
  }
}
{% endhighlight %}
