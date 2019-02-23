---
layout: post
title:  "Some C++ Snippets"
date:   2015-11-10 15:00:00
tags: [c++, snippets]
categories: Resources
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

### 2. stringstream进行数据类型转换
比sprintf等转换更简单，安全，方便 
[reference](https://blog.csdn.net/puppylpg/article/details/51260100)

{% highlight C++ scripts %}
#include <sstream>
template<class out_type,class in_value>
out_type convert(const in_value& t)
{
    std::stringstream stream;
    out_type result;        //这里存储转换结果
    
    stream << t;            //向流中传值
    stream >> result;       //向result中写入值
    return result;
}
// 使用
double outv;
std::string inv = "2.34";
outv = convert<double>(inv);  // Note: 模版参数可以省略第二个string，因为可以推断出来
std::string outv2 = convert<std::string>(outv);  // 2.34
// stringstream有临时缓冲区，可以一直往里写东西，最后统一写入文件，这样效率高
ofstream ofile("output.txt");
ostringstream oss;
oss << "blablabla" << endl;  // 多次写入
ofile << oss.str();  // 一次性写入文件
// 多次使用时，需要清空之前的内容时
oss.clear();  // 清空标志位
oss.str("");  // 清空内容，设置为空
{% endhighlight %}
另，高级用法参考[C++ 之定制输入输出流](http://kaiyuan.me/2017/06/22/custom-streambuf/)

### 3. 文件夹和文件操作
{% highlight C++ scripts %}
std::string findDirRecursively(const std::string &dirToFind, std::string curDir) {
    struct dirent *ptr;
    DIR *dir;
    while (1) {                                         // find the dir upwords
        if ((dir = opendir(curDir.c_str())) == NULL) {  // open the dir
            std::cerr << "Error open dir: " << curDir << std::endl;
            break;
        }
        while ((ptr = readdir(dir)) != NULL) {
            if ((ptr->d_type == 4) && (std::string(ptr->d_name) == dirToFind)) {
                // std::cout << "Find " << dirToFind << " in " << curDir << std::endl;
                return curDir + dirToFind + std::string("/");
            }
        }
        closedir(dir);
        curDir = std::string("../") + curDir;
    }
    return std::string();
}
{% endhighlight %}
