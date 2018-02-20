---
title: 【OpenCvTutorials3.1中文翻译】- 使用OpenCV添加（混合）两个图像
date: 2018-02-05 15:50:32
tags: [OpenCvTutorials3.1中文翻译]
categories: [原创]
thumbnail:  https://docs.opencv.org/3.1.0/Adding_Images_Tutorial_Result_Big.jpg
toc: true
---
<!-- Go to www.addthis.com/dashboard to customize your tools -->
<script type="text/javascript" src="//s7.addthis.com/js/300/addthis_widget.js#pubid=ra-59cb1e1aae525c27"></script>
### [Adding (blending) two images using OpenCV](https://docs.opencv.org/3.1.0/d5/dc4/tutorial_adding_images.html)

#### Goal
在本教程中，您将学习：
* 什么是线性混合以及为什么它是有用的;
* 如何使用[cv::addWeighted](https://docs.opencv.org/3.1.0/d2/de8/group__core__array.html#gafafb2513349db3bcff51f54ee5592a19)添加两个图像

<!-- more -->

#### 原理
##### Note:

下面的解释属于[计算机视觉：理查德Szeliski](http://szeliski.org/Book/)算法和应用程序

从我们以前的教程中，我们已经知道一些像素运算符。 一个有趣的二元（双输入）算子是线性混合算子：

![公式1](http://otneosm59.bkt.clouddn.com/18_2_5_06.png)

通过改变 `α0 → 1`

#### Code

像往常一样，在不那么冗长的解释之后，我们来看代码：

```C++
#include <opencv2/opencv.hpp>
#include <iostream>
using namespace cv;
int main( int argc, char** argv )
{
 double alpha = 0.5; double beta; double input;
 Mat src1, src2, dst;
 std::cout<<" Simple Linear Blender "<<std::endl;
 std::cout<<"-----------------------"<<std::endl;
 std::cout<<"* Enter alpha [0-1]: ";
 std::cin>>input;
 if( input >= 0.0 && input <= 1.0 )
   { alpha = input; }
 src1 = imread("../../images/LinuxLogo.jpg");
 src2 = imread("../../images/WindowsLogo.jpg");
 if( !src1.data ) { printf("Error loading src1 \n"); return -1; }
 if( !src2.data ) { printf("Error loading src2 \n"); return -1; }
 namedWindow("Linear Blend", 1);
 beta = ( 1.0 - alpha );
 addWeighted( src1, alpha, src2, beta, 0.0, dst);
 imshow( "Linear Blend", dst );
 waitKey(0);
 return 0;
}
```
#### 解释

1、由于我们要执行：
![公式2](http://otneosm59.bkt.clouddn.com/18_2_5_05.png)


我们需要两个源图像:
![公式3](http://otneosm59.bkt.clouddn.com/18_2_5_04.png)

```C++
src1 = imread("../../images/LinuxLogo.jpg");
src2 = imread("../../images/WindowsLogo.jpg");
```
##### warning
由于我们添加了src1和src2，它们必须具有相同的大小（宽度和高度）和类型。

2、现在我们需要生成g（x）图像。 为此，可以非常方便使用`add_weighted：addWeighted`函数：
因为[cv :: addWeighted](https://docs.opencv.org/3.1.0/d2/de8/group__core__array.html#gafafb2513349db3bcff51f54ee5592a19)产生：
![公式4](http://otneosm59.bkt.clouddn.com/18_2_5_03.png)


在这种情况下，`gamma`是参数0.0

3、创建窗口，显示图像并等待用户结束程序。

#### Result

![Result](https://docs.opencv.org/3.1.0/Adding_Images_Tutorial_Result_Big.jpg)