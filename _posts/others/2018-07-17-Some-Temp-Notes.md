---
layout: post
title:  "Some Temp Notes"
date:   2018-07-17 10:30:57
categories: Others
---

### 1. C++ 11 Clock
{% highlight C++ %}
inline double getTime() {
	return std::chrono::duration_cast<std::chrono::milliseconds>(std::chrono::high_resolution_clock::now().time_since_epoch()).count();
}
std::this_thread::sleep_for(std::chrono::milliseconds(1000));  // replace for usleep
clock_gettime(CLOCK_THREAD_CPUTIME_ID, &time);
{% endhighlight %}

### 2. [Winsock2.h Windows.h WIN32_LEAN_AND_MEAN冲突](https://blog.csdn.net/gongluck93/article/details/78854889)
[ref](https://blog.csdn.net/freefalcon/article/details/1374733)
{% highlight C++ %}
#define WIN32_LEAN_AND_MEAN
#include <windows.h>
{% endhighlight %}

### 3. 可变参数宏
{% highlight C++ %}
#define Debug(...) printf(__VA_ARGS__)
Debug(“Y = %d\n”, y);
// 编译器会把宏的调用替换成：
printf(“Y = %d\n”, y);
// Debug()是一个可变参数宏，能在每一次调用中传递不同数目的参数：
Debug(“test”); // 一个参数
{% endhighlight %}

### 4. SSE
Note: 用gcc编译时，如果报错指令not found，需要添加编译选项：-msse4.1
{% highlight C++ %}
if (inputs.size() == 2) {  // SSE
    unsigned char* inputPtr0 = (unsigned char*)inputs[0];
    unsigned char* inputPtr1 = (unsigned char*)inputs[1];
    const int blockSize = 16; // 16 * 8bit
    int i = 0;
    for (; i < width*height*storageChannel - blockSize; i += blockSize) {
        __m128i m1 = _mm_loadu_si128((__m128i*)&inputPtr0[i]);
        __m128i m2 = _mm_loadu_si128((__m128i*)&inputPtr1[i]);
        __m128i m3 = _mm_avg_epu8(m1, m2);  // TODO: diff, round not floor
        _mm_storeu_si128((__m128i*)&dst[i], m3);
    }
    for (; i < width*height*storageChannel; i++) {
        dst[i] = clampInt((unsigned int)inputPtr0[i] + inputPtr1[i], 0, 255);
    }
    break;
}
{% endhighlight %}

### 6. cmake install, cmake相关
https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#informational-expressionsinstall(FILES $<TARGET_PDB_FILE:kscnnrenderlib> DESTINATION lib/${CMAKE_ARCH}/ OPTIONAL)
### 7. c++ vector resize  old elements
### 8. rc资源文件，打包version信息
* 在Visual Studio中，可以给某个project，右键，New一个Resource，比如选择Version，填写好版本号，这样编译的时候会吧version信息打包到dll或者exe里。
* 具体生效的文件是project.rc文件(可能还有个resource.h,可以把内容拷贝到rc文件头部)，加到CMakeList即可
### 9. shared ptr用法问题
{% highlight C++ %}
class A {
public:
  int val = 100;
  A() {
    std::cout << "hi";
  }
  ~A() {
    std::cout << "111l";
  }
};
typedef std::vector<std::shared_ptr<A>*> PtrVector;
typedef std::vector<std::shared_ptr<A>> PtrVector1;
int main(int argc, char *argv[])
{
  PtrVector v(1);
  {
    std::shared_ptr<A> p1(new A);
    v[0] = &p1;
  }
  {
    std::shared_ptr<A> p2 = *(v[0]);
    PtrVector1 v2(1);
    v2[0] = p2;
  }
  std::cout<<v.size();
    QApplication a(argc, argv);
    qweb w;
    w.show();
    return a.exec();
}
{% endhighlight %}
### 10. ubuntu系统，查找安装软件包
{% highlight Bash shell scripts %}
sudo aptitude search binutils
sudo aptitude install binutils-2.26  # 安装特定版本ld链接程序
# 或者，用 apt-get, 在之前可以update upgrade一下
sudo apt-cache search binutils
sudo apt-get install binutils-2.26
{% endhighlight %}
### 11. Upsample
### 12. ffprobe test.mp4  // 查看视频编码信息，如h264 (High), yuvj420p(pc), 1080x1920, 5344 kb/s, 27.91 fps
ffmpeg -i src0.mov -c:v libx264 -crf 27 -preset slow -an dst0.mp4  // 压缩视频大小
[ffmeg doc](https://ffmpeg.org/ffmpeg.html)
[ffmpeg flags](https://gist.github.com/tayvano/6e2d456a9897f55025e25035478a3a50)
### 13. OneEureFilter  KalmanFilter
[](https://github.com/akshaychawla/1D-Kalman-Filter)
[](https://github.com/bachagas/Kalman)
### 14. g++ -pie -fPIE  -I ./include/ -std=gnu++11 -L libs/ -ldepend bench.cpp
Use -L point to the library, and -l to the libdepend.so
### 15. MobileNetV2:中间通道更多的Depthwise的Residual Block，并且去掉最后的ReLU
### 16. softmax加速：1. e指数近似方法  2. 限制到-4～4，然后量化到0～255，查表计算
### 17. Instance Normalization: 每个例子的每个通道进行normalization,在做style transfer的时候，只要计算单个图中每个channel的统计量，跟其他图无关。否则结果有些奇怪，比如人脸有突兀的阴影。
### 18. nm -A (-D) libycnn2.so | grep "Android"： 查找符号表
### 19. [execstack](https://stackoverflow.com/questions/23276488/why-is-execstack-required-to-execute-code-on-the-heap/23277109)用来操作是否允许堆上执行代码
* execstack -s(set) -c(clear) -q(query) the flag
* execstack is On by default, -z noexecstack in gcc flag will OFF it
