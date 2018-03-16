---
title: 【OpenCvTutorials3-1中文翻译】- 平滑图像
toc: true
thumbnail: 'http://image.little-rocket.cn/Smoothing_Tutorial_theory_gaussian_0.jpg'
tags:
  - OpenCvTutorials3.1中文翻译
categories:
  - zhangcaocao
abbrlink: 9300c253
date: 2018-03-12 17:09:04
---

<!-- Go to www.addthis.com/dashboard to customize your tools -->
<script type="text/javascript" src="//s7.addthis.com/js/300/addthis_widget.js#pubid=ra-59cb1e1aae525c27"></script>

<script>
(function(){
    var bp = document.createElement('script');
    var curProtocol = window.location.protocol.split(':')[0];
    if (curProtocol === 'https') {
        bp.src = 'https://zz.bdstatic.com/linksubmit/push.js';
    }
    else {
        bp.src = 'http://push.zhanzhang.baidu.com/push.js';
    }
    var s = document.getElementsByTagName("script")[0];
    s.parentNode.insertBefore(bp, s);
})();
</script>

### [Smoothing Images](https://docs.opencv.org/3.1.0/dc/dd3/tutorial_gausian_median_blur_bilateral_filter.html)

#### Goal

在本教程中，您将学习如何使用OpenCV函数应用各种线性滤波器来平滑图像，比如说：

[cv::blur](https://docs.opencv.org/3.1.0/d4/d86/group__imgproc__filter.html#ga8c45db9afe636703801b0b2e440fce37)
[cv::GaussianBlur](https://docs.opencv.org/3.1.0/d4/d86/group__imgproc__filter.html#gaabe8c836e97159a9193fb0b11ac52cf1)
[cv::medianBlur](https://docs.opencv.org/3.1.0/d4/d86/group__imgproc__filter.html#ga564869aa33e58769b4469101aac458f9)
[cv::bilateralFilter](https://docs.opencv.org/3.1.0/d4/d86/group__imgproc__filter.html#ga9d7064d478c95d60003cf839430737ed)
<!-- more -->

#### Theory

* Note:

The explanation below belongs to the book [Computer Vision: Algorithms and Applications by Richard Szeliski](http://szeliski.org/Book/) and to LearningOpenCV .. container:: enumeratevisibleitemswithsquare

* 平滑(Smoothing)，也称为模糊(blurring)，是一种简单且经常使用的图像处理操作。
* 平滑有很多原因。 在本教程中，我们将重点关注平滑以减少噪音（其他用途将在以下教程中看到）。
* 要执行平滑操作，我们将对图像应用滤波器。 最常见的滤波器类型是线性的，其中输出像素值(即 $g(i，j)$)被确定为输入像素值（即 $f(i + k，j + 1))$的加权和：

$$g(i,j) = \sum_{k,l} f(i+k, j+l) h(k,l)$$

<br>
$h(k,l)$ 被称为内核，它不过是滤波器的系数。它有助于将滤波器可视化为系数在图像上滑动的窗口。

* 有很多类型的过滤器，在这里我们会提到最常用的：


##### Normalized Box Filter(归一化)

* 这个过滤器是最简单的！ 每个输出像素是其内核邻居的平均值（所有这些都是以相同的权重作出贡献的）
* 内核如下：

![公式截图1](http://image.little-rocket.cn/18_03_12_01.PNG)

##### Gaussian Filter

* 可能是最有用的过滤器（尽管不是最快的）。滤波是通过将输入数组中的每个点与高斯内核进行卷积然后将它们相加以产生输出数组来完成的。
* 为了让图像更清晰，请记住一维高斯内核的是怎样的？

![一阶高斯](http://image.little-rocket.cn/Smoothing_Tutorial_theory_gaussian_0.jpg)

假设图像是1D，您可以注意到位于中间的像素将具有最大的权重。 其邻居的权重随着它们与中心像素之间的空间距离的增加而减小。


* Note:
请记住，二维高斯可以表示为：

![公式截图2](http://image.little-rocket.cn/18_03_12_02.PNG)

其中$μ$是平均值（峰值），$σ$表示方差（每个变量$x$和$y$）

##### Median Filter

中值滤波器贯穿信号的每个元素（在这种情况下是图像），并用每个像素的相邻像素的中值（位于评估像素周围的正方形邻域中）替换每个像素。

##### Bilateral Filter(双边滤波)

* 到目前为止，我们已经解释了一些滤波器，其主要目标是平滑输入图像。 但是，有时滤波器不仅会消除噪音，还会消除边缘。 为了避免这种情况（至少在某种程度上），我们可以使用双边滤波器。
* 以与高斯滤波器类似的方式，双边滤波器也考虑分配给它们中的每一个相邻像素一个权重。 这些权重有两个分量，第一个是高斯滤波器使用的相同权重。 第二部分考虑了相邻像素与评估像素之间强度的差异。
* 关于更详细的解释你可以查看这个[链接](http://homepages.inf.ed.ac.uk/rbf/CVonline/LOCAL_COPIES/MANDUCHI1/Bilateral_Filtering.html)


#### Code

* 这个程序做什么？
    - 加载一张图
    - 应用4种不同类型的过滤器（在 Theory 中解释了的）并顺序显示过滤后的图像

* Downloadable code: Click [here](https://github.com/Itseez/opencv/tree/master/samples/cpp/tutorial_code/ImgProc/Smoothing.cpp)
* 代码一览:

```C++
#include "opencv2/imgproc.hpp"
#include "opencv2/highgui.hpp"
using namespace std;
using namespace cv;
int DELAY_CAPTION = 1500;
int DELAY_BLUR = 100;
int MAX_KERNEL_LENGTH = 31;
Mat src; Mat dst;
char window_name[] = "Filter Demo 1";
int display_caption( char* caption );
int display_dst( int delay );
/*
 * function main
 */
 int main( int argc, char** argv )
 {
   namedWindow( window_name, WINDOW_AUTOSIZE );
   src = imread( "../images/lena.jpg", 1 );
   if( display_caption( "Original Image" ) != 0 ) { return 0; }
   dst = src.clone();
   if( display_dst( DELAY_CAPTION ) != 0 ) { return 0; }
   if( display_caption( "Homogeneous Blur" ) != 0 ) { return 0; }
   for ( int i = 1; i < MAX_KERNEL_LENGTH; i = i + 2 )
       { blur( src, dst, Size( i, i ), Point(-1,-1) );
         if( display_dst( DELAY_BLUR ) != 0 ) { return 0; } }
    if( display_caption( "Gaussian Blur" ) != 0 ) { return 0; }
    for ( int i = 1; i < MAX_KERNEL_LENGTH; i = i + 2 )
        { GaussianBlur( src, dst, Size( i, i ), 0, 0 );
          if( display_dst( DELAY_BLUR ) != 0 ) { return 0; } }
 if( display_caption( "Median Blur" ) != 0 ) { return 0; }
 for ( int i = 1; i < MAX_KERNEL_LENGTH; i = i + 2 )
         { medianBlur ( src, dst, i );
           if( display_dst( DELAY_BLUR ) != 0 ) { return 0; } }
 if( display_caption( "Bilateral Blur" ) != 0 ) { return 0; }
 for ( int i = 1; i < MAX_KERNEL_LENGTH; i = i + 2 )
         { bilateralFilter ( src, dst, i, i*2, i/2 );
           if( display_dst( DELAY_BLUR ) != 0 ) { return 0; } }
 display_caption( "End: Press a key!" );
 waitKey(0);
 return 0;
 }
 int display_caption( char* caption )
 {
   dst = Mat::zeros( src.size(), src.type() );
   putText( dst, caption,
            Point( src.cols/4, src.rows/2),
            FONT_HERSHEY_COMPLEX, 1, Scalar(255, 255, 255) );
   imshow( window_name, dst );
   int c = waitKey( DELAY_CAPTION );
   if( c >= 0 ) { return -1; }
   return 0;
  }
  int display_dst( int delay )
  {
    imshow( window_name, dst );
    int c = waitKey ( delay );
    if( c >= 0 ) { return -1; }
    return 0;
  }
```


#### Explanation(解释)

##### 1、让我们来看看只涉及平滑过程的OpenCV函数，因为掌握这个之后其余的内容就已经知道了。

##### 2、 Normalized Block Filter:

OpenCV提供[cv::blur](https://docs.opencv.org/3.1.0/d4/d86/group__imgproc__filter.html#ga8c45db9afe636703801b0b2e440fce37)函数来执行此滤波器的平滑处理。

```C++
for ( int i = 1; i < MAX_KERNEL_LENGTH; i = i + 2 )
    { blur( src, dst, Size( i, i ), Point(-1,-1) );
      if( display_dst( DELAY_BLUR ) != 0 ) { return 0; } }
```

我们指定了4个参数（更多细节，请参阅参考资料）：

* src：源图像
* dst：目标图像
* size（w，h）：定义要使用的内核的大小（宽度w像素和高度h像素）
* Point（-1，-1）：指示参考点（评估像素）相对于邻域的位置。 如果存在负值，那么内核的中心被认为是参考点。

##### 3、Gaussian Filter:

它由[cv::GaussianBlur](https://docs.opencv.org/3.1.0/d4/d86/group__imgproc__filter.html#gaabe8c836e97159a9193fb0b11ac52cf1)函数执行：

```C++
for ( int i = 1; i < MAX_KERNEL_LENGTH; i = i + 2 )
    { 
        GaussianBlur( src, dst, Size( i, i ), 0, 0 );
      if( display_dst( DELAY_BLUR ) != 0 ) { return 0; } 
    }
```

这里我们指定了4个参数（更多细节，请参阅参考资料）：

- src：源图像
- dst：目标图像
- size（w，h）：要使用的内核的大小（the neighbors to be considered）。 m和h必须是奇数和正数，否则将使用σx和σy参数计算大小。
- σx：x中的标准差。 写0意味着σx是使用内核大小计算的。
- σy：y中的标准差。 写0意味着使用内核大小来计算σy。


##### 4、Median Filter:

该过滤器由[cv::medianBlur](https://docs.opencv.org/3.1.0/d4/d86/group__imgproc__filter.html#ga564869aa33e58769b4469101aac458f9)函数提供：

```C++
for ( int i = 1; i < MAX_KERNEL_LENGTH; i = i + 2 )
    { medianBlur ( src, dst, i );
      if( display_dst( DELAY_BLUR ) != 0 ) { return 0; } }
```

我们使用了3个参数：

* Src:源图像
* dst:目标图像
* i：内核的大小（只有一个，因为我们使用一个方形窗口）。必须是奇数。


##### 5、Bilateral Filter：

[cv::bilateralFilter](https://docs.opencv.org/3.1.0/d4/d86/group__imgproc__filter.html#ga9d7064d478c95d60003cf839430737ed)

```C++
for ( int i = 1; i < MAX_KERNEL_LENGTH; i = i + 2 )
    { bilateralFilter ( src, dst, i, i*2, i/2 );
      if( display_dst( DELAY_BLUR ) != 0 ) { return 0; } }
```

我们使用了5个参数：

* src:源图像
* dst:目标图像
* d： 每个像素邻域的直径。
* $\sigma_{Color}$ : 色彩空间中的标准偏差。
* $\sigma_{Space}$ : 坐标空间中的标准偏差（以像素为单位）


#### Results
* 该代码打开一个图像（在本例中为lena.jpg），并在所解释的4个过滤器的效果下显示。
* 以下是使用medianBlur平滑图像的快照：

![结果快照](http://image.little-rocket.cn/Smoothing_Tutorial_Result_Median_Filter.jpg)
