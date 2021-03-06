---
layout: post
title:  "Leetcode Shell Problems"
date:   2015-12-01 11:30:00
tags: [algorithm, leetcode, shell, Tenth Line, Valid Phone Numbers, Word Frequency, Transpose File]
categories: Algorithm
---

> Leetcode上增加了4道Shell题目，进行总结

### 1. [Tenth Line](https://leetcode.com/problems/tenth-line/)
```
How would you print just the 10th line of a file?
For example, assume that file.txt has the following content:
Line 1
Line 2
Line 3
Line 4
Line 5
Line 6
Line 7
Line 8
Line 9
Line 10

Your script should output the tenth line, which is:
Line 10
```
a. `head -10 file.txt | tail -1`
   错误: 若文件只有9行，会打印出Line 9，但是应该不打印
   修改：tail -n +10 file.txt | head -n1 (+10代表从正数10行开始，如果没有就不输出)
   同理：若打印倒数第10行：head -n -10 404.html | tail -1

b. if语句实现相同功能：

{% highlight Bash shell scripts %}
if [ \`wc -l file.txt | awk '{print $1}'\` -gt 9 ]; then
	head -10 file.txt | tail -1
fi
{% endhighlight %}

c. `sed -n '10p' file.txt`
不带-n的话，不仅打印匹配行，还接着输出file的全部内容。

d. `awk 'NR == 10' file.txt` Number of Field, 默认是换行符，就是第几行

### 2. [Valid Phone Numbers](https://leetcode.com/problems/valid-phone-numbers/)
```
Given a text file file.txt that contains list of phone numbers (one per line), write a one liner bash script to print all valid phone numbers.
You may assume that a valid phone number must appear in one of the following two formats: (xxx) xxx-xxxx or xxx-xxx-xxxx. (x means a digit)
You may also assume each line in the text file must not contain leading or trailing white spaces.

For example, assume that file.txt has the following content:
	987-123-4567
	123 456 7890
	(123) 456-7890
Your script should output the following valid phone numbers:
	987-123-4567
	(123) 456-7890
```
几种解法：
{% highlight Bash shell scripts %}
grep -P '^(\d{3}-|\(\d{3}\) )\d{3}-\d{4}$' file.txt  # grep的perl正则表达式扩展（可以用[0-9]替换\d）
sed -n -r '/^([0-9]{3}-|\([0-9]{3}\) )[0-9]{3}-[0-9]{4}$/p' file.txt  # sed的-r正则扩展，与上个区别：不支持\d
awk '/^([0-9]{3}-|\([0-9]{3}\) )[0-9]{3}-[0-9]{4}$/' file.txt  # 基本跟sed一模一样
{% endhighlight %}

### 3. [Word Frequency](https://leetcode.com/problems/word-frequency/)
```
Write a bash script to calculate the frequency of each word in a text file words.txt.
For simplicity sake, you may assume:

words.txt contains only lowercase characters and space ' ' characters.
Each word must consist of lowercase characters only.
Words are separated by one or more whitespace characters.
For example, assume that words.txt has the following content:
	the day is sunny the the
	the sunny is is
Your script should output the following, sorted by descending frequency:
	the 4
	is 3
	sunny 2
	day 1
Note:
Don't worry about handling ties, it is guaranteed that each word's frequency count is unique
```
解法：
{% highlight Bash shell scripts %}
# 关键：tr -s，字符替换，-s删除文件中重复的字符只保留一个
cat words.txt | tr -s " " "\n" | sort | uniq -c | sort -rn | awk '{print $2 " " $1}'
{% endhighlight %}

### 4. [Transpose File](https://leetcode.com/problems/transpose-file/)
```
Given a text file file.txt, transpose its content.
You may assume that each row has the same number of columns and each field is separated by the ' ' character.
For example, if file.txt has the following content:
	name age
	alice 21
	ryan 30

Output the following:
	name alice ryan
	age 21 30
```
解法：
{% highlight Bash shell scripts %}
awk '{ 
    if(NR==1){    # 当前处理到了第几行，也即初始化 列数组
        for(i=1;i<=NF;i++){
            arr[i]=$i
        }
    } 
    else{
        for(i=1;i<=NF;i++){
            arr[i]=arr[i]" "$i
        }
    } 
} 
END {
    for(i=1;i<=NF;i++){
        print arr[i]
    }
}' file.txt
{% endhighlight %}
