---
layout: post
title:  "Basics on Dynamic Programming(DP) 8 - Others"
date:   2016-03-22 18:30:00
tags: [algorithm, leetcode, dynamic programming, dp]
categories: Algorithm
---

> 其他典型问题

### 1. n个人排名，允许并列，共有多少种排名结果

```
分析：
假设n个人，排出了m个名次，有f(n,m)种结果（1 <=m <=n） 
当m=1，f(n,m)=1 
当n<m，f(n,m)=0 
当1<m<=n，f(n,m)= f(n-1,m)*m + f(n-1,m-1)*m :
假设n-1个人，排出了m个名次；新来1人，与前面某名次并列，有f(n-1,m)*m种结果 
假设n-1个人，排出了m个名次；新来1人，与前面名次都不并列，有f(n-1,m-1)*m种结果 
```

综合上述，递推式: 

```
f(n,m) = 0 , n <m||m <1 
f(n,m) = 1 , 1=m <=n 
f(n,m) = (f(n-1,m) + f(n-1,m-1))*m 1 <m <=n 

n个人的排名就是f(n,1)+f(n,2)+f(n,3)+...+f(n,n) 
```

