---
layout: post
title:  "Equalize Hist & White Balance"
date:   2018-07-08 16:00:00
tags: [opencv, equalize, hist, feature]
categories: CV
---

> [直方图均衡化 wiki](https://zh.wikipedia.org/zh-cn/直方图均衡化)

> [白平衡](http://www.ipol.im/pub/art/2011/llmps-scb/)

### 1. 基本概念
1. 增加图像的全局对比度，增加细节，对于背景和前景都太亮或者太暗的图像非常有用
2. 算法直观，通过查表得到变化前后的像素值，是可逆操作
3. 缺点：对所有像素进行变换，可能会增加噪声的对比度，或者降低有用信号的对比度
4. 注意：不要对彩色图片的3个通道都进行均衡化，否则会造成颜色失真，图像不和谐。一般只对yuv空间的y亮度通道进行均衡

### 2. 算法实现O(N)
{% highlight C++ %}
// onlyFirstCh: 是否只处理输入图片的第一个channel
void equalize_hist(cv::Mat &mat, bool onlyFirstCh = true) {
  int chNum = mat.channels();
  std::vector<std::vector<int>> hists(onlyFirstCh ? 1 : chNum, std::vector<int>(256, 0));
  // 统计颜色bin
  int jth = onlyFirstCh ? 1 : chNum;
  for (int y = 0; y < mat.rows; ++y) {
    uchar *ptr = mat.ptr<uchar>(y);
    for (int x = 0; x < mat.cols; ++x) {
      for (int j = 0; j < jth; ++j) {
        hists[j][ptr[x * chNum + j]] += 1;
      }
    }
  }
  // cumulative hist 累计直方图
  float total = (float)mat.cols * mat.rows;
  int ith = onlyFirstCh ? 1 : chNum;
  for (int i = 0; i < ith; ++i) {
    for (int j = 0; j < 255; ++j) {
      hists[i][j + 1] += hists[i][j];
      hists[i][j] = round(hists[i][j] / total * 255); // 转变成概率*255
    }
    hists[i][255] = round(hists[i][255] / total * 255);  // 转变成概率*255
  }
  // 计算输出
  for (int y = 0; y < mat.rows; ++y) {
    uchar *ptr = mat.ptr<uchar>(y);
    for (int x = 0; x < mat.cols; ++x) {
      for (int j = 0; j < jth; ++j) {
        int val = ptr[x * chNum + j];
        ptr[x * chNum + j] = static_cast<uchar>(hists[j][val]); // 查表
      }
    }
  }
}
// OpenCV库，也有类似的实现，和上述代码效果一致
std::vector<cv::Mat> channels;
cv::cvtColor(src, src, CV_BGR2YUV);
cv::split(hist_cv, channels);
// 只处理 Y 通道
cv::equalizeHist(channels[0], channels[0]);
cv::merge(channels, src);
cv::cvtColor(src, src, CV_YUV2BGR);
{% endhighlight %}

### 3. 算法对比：[白平衡](http://www.ipol.im/pub/art/2011/llmps-scb/)
1. 该算法的论文，[参考链接](http://www.ipol.im/pub/art/2011/llmps-scb/)
2. 该简单的白平衡算法，基于假设：图片中RGB通道的最高值pixel的颜色原始就是白色(255,255,255)，最低值就是黑色(0,0,0)
3. 思想：去掉图像中某些过亮/暗的躁点(如前后各去掉1%)，然后将所有的pixel线性拉伸到0～255
4. 用途：某些图片中本身有白色，但是由于在某种灯光环境下(比如黄色灯泡的屋子，KTV，黄昏等)，白色变成了其他颜色，可以通过白平衡矫正

O(N)算法实现：
{% highlight C++ %}
void balance_white(cv::Mat &mat, cv::Mat mask = cv::Mat()) {
  if (mask.empty()) {  // mask作为掩码
    mask.create(mat.rows, mat.cols, CV_8U);
    mask.setTo(cv::Scalar(1));
  }
  double discard_ratio = 0.01;
  int hists[3][256];
  memset(hists, 0, 3 * 256 * sizeof(int));

  for (int y = 0; y < mat.rows; ++y) {
    uchar *ptr = mat.ptr<uchar>(y);
    uchar *bounds_ptr = mask.ptr<uchar>(y);
    for (int x = 0; x < mat.cols; ++x) {
      for (int j = 0; j < 3; ++j) {
        if (bounds_ptr[x] != 0) {
          hists[j][ptr[x * 3 + j]] += 1;
        }
      }
    }
  }
  // cumulative hist
  int total = sum(mask)[0];
  int vmin[3], vmax[3];
  for (int i = 0; i < 3; ++i) {
    for (int j = 0; j < 255; ++j) {
      hists[i][j + 1] += hists[i][j];
    }
    vmin[i] = 0;
    vmax[i] = 255;
    while (hists[i][vmin[i]] < discard_ratio * total) vmin[i] += 1;
    while (hists[i][vmax[i]] > (1 - discard_ratio) * total) vmax[i] -= 1;
    if (vmax[i] < 255 - 1) vmax[i] += 1;
  }
  // 线性拉伸
  for (int y = 0; y < mat.rows; ++y) {
    uchar *ptr = mat.ptr<uchar>(y);
    uchar *bounds_ptr = mask.ptr<uchar>(y);
    for (int x = 0; x < mat.cols; ++x) {
      for (int j = 0; j < 3; ++j) {
        if (bounds_ptr[x]) {
          int val = ptr[x * 3 + j];
          if (val < vmin[j]) val = vmin[j];
          if (val > vmax[j]) val = vmax[j];
          ptr[x * 3 + j] = static_cast<uchar>((val - vmin[j]) * 255.0 / (vmax[j] - vmin[j]));
        }
      }
    }
  }
}
{% endhighlight %}

### 4. 一些结果
1. equalize hist的好的结果：
![cv_face_1](/res/cv_face_1.png)
图中，第三张equal hist，把皮肤变得很亮，整体图片对比度提升

2. white balance好的结果：
![cv_balance_red](/res/cv_balance_red.png)
图中右上白平衡后，白色的盘子被矫正，整体图片和谐。左下角equal hist只是把整体亮度提升，也产生了噪点，右下颜色很奇怪

下图类似：
![cv_face_4](/res/cv_face_4.png)

3. 如果3个通道一起做，会产生异常颜色
![cv_tree](/res/cv_tree.png)
图中第二张白平衡没有明显变化，第三张草地的颜色变得清晰，细节更丰富，但是第四张出现了自然界中不存在的树叶的颜色

4. equalize hist有时会产生噪点
![cv_face_6](/res/cv_face_6.png)
图中，第三张产生了很多噪点

5. equalize hist有时会丢失细节，y通道过亮，导致颜色细节丢失
![cv_face_8](/res/cv_face_8.png)
图中，第三张下部分，杯子的花纹过亮，细节花纹颜色丢失
