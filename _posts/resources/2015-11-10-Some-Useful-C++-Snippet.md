---
layout: post
title:  "Some C++ Snippets"
date:   2015-11-10 15:00:00
tags: [c++, snippets]
categories: resources
---

### 1. Sort a vector and Get the Index
vector<float> values存储一个list, 得到list中从大到小的index序列
{% highlight C++ scripts %}
struct IdxCompare {
    const std::vector<float>& target;
    IdxCompare(const std::vector<float>& target): target(target) {}  // 导入要排序的list
    bool operator()(int a, int b) const { return target[a] > target[b]; }  // 重载这个
};
vector<size_t> indices(values.size());  // store the sorted index
for(int i=0; i<values.size(); i++)
    indices[i] = i;
sort(indices.begin(), indices.end(), IdxCompare(values));  // 传进去
{% endhighlight %}
如果是C++11(>=g++4.8)，可以调用Lamdba表达式
{% highlight C++ scripts %}
vector<size_t> indices(values.size());
std::iota(begin(indices), end(indices), stat，ic_cast<size_t>(0));
std::sort(
	begin(indices), end(indices),
	[&](size_t a, size_t b) { return values[a] > values[b]; }  // 非常简洁
);
{% endhighlight %}