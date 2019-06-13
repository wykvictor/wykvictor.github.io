---
layout: post
title:  "Approximately exp: 近似e指数"
date:   2019-06-08 16:00:00
tags: [approximately, exp]
categories: Tech
---

> [参考](https://www.zhihu.com/question/51026869)

### 1. C++实现
{% highlight C++ %}
union exp_appro_struct_32x1{
    int a;
    float b;
};
float ksf_exp(float x){
   exp_appro_struct_32x1 res_struct;
   res_struct.a = int(-12102203*(-x)) + 1065353216;
   return res_struct.b;
}
// example
for(int i=0; i<40; i++)
    printf("%f ", ycnn2::ksf_exp(i/10.));
/* output
1.000000 1.144269 1.288539 1.432809 1.577078 1.721347 1.865617 2.019773 2.308312 2.596851 2.885390 3.173929 3.462468 3.751007 4.079092 4.656170 5.233249 5.810327 6.387403 6.964482 7.541560 8.237276 9.391434 10.545588 11.699745 12.853901 14.008055 15.162212 16.632736 18.941048 21.249359 23.557671 25.865990 28.174294 30.482613 33.581848 38.198456 42.815094 47.431717 52.048340
*/
{% endhighlight %}

### 2. Python实现
{% highlight Python scripts %}
class ExpAppro():
    k = torch.zeros(12)
    b = torch.zeros(12)
    t = np.arange(-6., 7., 1.)
    y_p = 2. ** t
    y_p_diff = y_p[1:] - y_p[:-1]
    bb = y_p[1:] - y_p_diff * t[1:]
    kk = y_p_diff / np.log(2)
    k, b = (torch.from_numpy(kk)).float(), (torch.from_numpy(bb)).float()

    @staticmethod
    def exp(x):
        x = F.hardtanh(x, -4, 4)
        x_ = x.unsqueeze(-1)
        x_seg = (x_) * Variable(ExpAppro.k, requires_grad=False) + Variable(ExpAppro.b, requires_grad=False)
        x_exp, _ = torch.max(x_seg, dim=-1)
        return x_exp
torch.set_printoptions(precision=6)
ExpAppro.exp(aa)
'''
output
tensor([ 1.000000,  1.144269,  1.288539,  1.432809,  1.577078,  1.721348,
         1.865617,  2.019773,  2.308312,  2.596851,  2.885390,  3.173929,
         3.462468,  3.751007,  4.079092,  4.656170,  5.233249,  5.810327,
         6.387403,  6.964482,  7.541560,  8.237276,  9.391434, 10.545588,
        11.699745, 12.853901, 14.008055, 15.162212, 16.632736, 18.941048,
        21.249359, 23.557671, 25.865990, 28.174294, 30.482613, 33.581848,
        38.198456, 42.815094, 47.431717, 52.048340])
'''
{% endhighlight %}

### 4. 正常exp
{% highlight Python scripts %}
np.exp(a)
array([ 1.        ,  1.10517092,  1.22140276,  1.34985881,  1.4918247 ,
        1.64872127,  1.8221188 ,  2.01375271,  2.22554093,  2.45960311,
        2.71828183,  3.00416602,  3.32011692,  3.66929667,  4.05519997,
        4.48168907,  4.95303242,  5.47394739,  6.04964746,  6.68589444,
        7.3890561 ,  8.16616991,  9.0250135 ,  9.97418245, 11.02317638,
       12.18249396, 13.46373804, 14.87973172, 16.44464677, 18.17414537,
       20.08553692, 22.19795128, 24.5325302 , 27.11263892, 29.96410005,
       33.11545196, 36.59823444, 40.44730436, 44.70118449, 49.40244911])
{% endhighlight %}
2种近似结果基本相同，和exp有些差距
