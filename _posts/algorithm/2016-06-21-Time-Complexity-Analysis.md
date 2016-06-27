---
layout: post
title:  "Time Complexity Analysis"
date:   2016-06-21 13:30:00
tags: [algorithm, time, complexity]
categories: Algorithm
---

### 1. Summary
![time_complex](http://7xno5y.com1.z0.glb.clouddn.com/time_complex.jpg)

### 2. Reduction
通过O(n)的时间，把规模为n的问题变为n/2：

![time_complex2](http://7xno5y.com1.z0.glb.clouddn.com/time_complex2.jpg)

通过O(n)的时间，把规模为n的问题变为(2 * n/2)：

![time_complex3](http://7xno5y.com1.z0.glb.clouddn.com/time_complex3.jpg)

### 3. Time Complexity in Coding Interview
* O(1) 极少
* O(logn) 几乎都是二分法
* O(√n) 几乎是分解质因数
* O(n) 高频
* O(nlogn) 一般都可能要排序
* O(n2) 数组，枚举，动态规划
* O(n3) 数组，枚举，动态规划
* O(2n) 与组合有关的搜索
* O(n!) 与排列有关的搜索 