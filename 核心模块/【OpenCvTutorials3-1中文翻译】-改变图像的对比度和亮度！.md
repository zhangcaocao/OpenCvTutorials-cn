---
title: 【OpenCvTutorials3.1中文翻译】- 改变图像的对比度和亮度！
date: 2018-02-05 17:19:16
tags: [OpenCvTutorials3.1中文翻译]
categories: [原创]
thumbnail:  https://docs.opencv.org/3.1.0/Basic_Linear_Transform_Tutorial_Result_big.jpg
toc: true
---
<!-- Go to www.addthis.com/dashboard to customize your tools -->
<script type="text/javascript" src="//s7.addthis.com/js/300/addthis_widget.js#pubid=ra-59cb1e1aae525c27"></script>
### [Changing the contrast and brightness of an image!](https://docs.opencv.org/3.1.0/d3/dc1/tutorial_basic_linear_transform.html)

#### Goal

在本教程中，您将学习如何：

* 访问像素值
* 用零初始化一个矩阵
* 了解[cv::saturate_cast](https://docs.opencv.org/3.1.0/db/de0/group__core__utils.html#gab93126370b85fda2c8bfaf8c811faeaf)做什么以及为什么它很有用
* 获取有关像素变换的一些很酷的信息

<!-- more -->
#### Theory

* Note:
下面的解释属于[计算机视觉：理查德Szeliski算法和应用程序](http://szeliski.org/Book/)

##### 图像处理

* 一般的图像处理操作符是一个获取一个或多个输入图像并产生输出图像的功能。
* 图像转换可以被看作是：
    * 点运算符（像素变换）
    * 邻域操作

##### 像素变换
* 在这种图像处理变换中，每个输出像素的值仅取决于相应的输入像素值（可能还有一些全局收集的信息或参数）。

* 这些例子的操作包括亮度和对比度调整以及颜色校正和变换。

##### 亮度和对比度调整

* 两个常用的点过程是乘上和加上一个常量：

> 译者注：
> 原文： “Two commonly used point processes are multiplication and >addition with a constant:”不好翻译， 可以结合下面的公式理解

![公式1](http://otneosm59.bkt.clouddn.com/18_2_5_08.png)

* 参数 α > 0β
* You can think of f(x)g(x)：

![公式2](http://otneosm59.bkt.clouddn.com/18_2_5_09.png)

where i j

#### Code
以下代码执行运算： 
![公式1](http://otneosm59.bkt.clouddn.com/18_2_5_08.png)
```C++
#include <opencv2/opencv.hpp>
#include <iostream>
using namespace cv;
double alpha; /*< Simple contrast control */
int beta;  /*< Simple brightness control */
int main( int argc, char** argv )
{
    Mat image = imread( argv[1] );
    Mat new_image = Mat::zeros( image.size(), image.type() );
    std::cout<<" Basic Linear Transforms "<<std::endl;
    std::cout<<"-------------------------"<<std::endl;
    std::cout<<"* Enter the alpha value [1.0-3.0]: ";std::cin>>alpha;
    std::cout<<"* Enter the beta value [0-100]: "; std::cin>>beta;
    for( int y = 0; y < image.rows; y++ ) {
        for( int x = 0; x < image.cols; x++ ) {
            for( int c = 0; c < 3; c++ ) {
                new_image.at<Vec3b>(y,x)[c] =
                saturate_cast<uchar>( alpha*( image.at<Vec3b>(y,x)[c] ) + beta );
            }
        }
    }
    namedWindow("Original Image", 1);
    namedWindow("New Image", 1);
    imshow("Original Image", image);
    imshow("New Image", new_image);
    waitKey();
    return 0;
}
```

#### 说明
1、 我们首先创建参数来保存α β
```C++
double alpha;
int beta;
```
2、我们使用[cv::imread](https://docs.opencv.org/3.1.0/d4/da8/group__imgcodecs.html#ga288b8b3da0892bd651fce07b3bbd3a56)加载图像，并将其保存在Mat对象中：

```C++
Mat image = imread( argv[1] );
```
3、现在，由于我们将对这个图像进行一些转换，我们需要一个新的Mat对象来存储它。 另外，我们希望这具有以下特点：
    * 初始像素值等于零
    * 与原始图像大小和类型相同
```C++
Mat new_image = Mat::zeros( image.size(), image.type() );
```
我们观察到[cv::Mat::zeros](https://docs.opencv.org/3.1.0/d3/d63/classcv_1_1Mat.html#a0b57b6a326c8876d944d188a46e0f556)返回一个基于`image.size（）`和`image.type（）`的Matlab风格的零初始化器.


4、现在，执行操作 `g（i，j）=α·f（i，j）+β`

```C++
for( int y = 0; y < image.rows; y++ ) {
    for( int x = 0; x < image.cols; x++ ) {
        for( int c = 0; c < 3; c++ ) {
            new_image.at<Vec3b>(y,x)[c] =
              saturate_cast<uchar>( alpha*( image.at<Vec3b>(y,x)[c] ) + beta );
        }
    }
}
```
注意以下几点：
    * 要访问图像中的每个像素，我们使用这个语法：image.at <Vec3b>（y，x）[c]其中y是行，x是列，c是R，G或B（0,1或2）。
    * 由于运算α⋅p（i，j）+βα⋅p（i，j）+β
> 译者注：
> 原文： "Since the operation α⋅p(i,j)+βα⋅p(i,j)+βα"后面多了个α。

#### 最后，我们创建窗口，并显示图像。

```C++
namedWindow("Original Image", 1);
namedWindow("New Image", 1);
imshow("Original Image", image);
imshow("New Image", new_image);
waitKey(0);
```
* Note:
我们可以简单地使用这个命令, 而不是使用for循环来访问每个像素：
```C++
image.convertTo(new_image, -1, alpha, beta);
```

其中[cv::Mat::convertTo](https://docs.opencv.org/3.1.0/d3/d63/classcv_1_1Mat.html#a3f356665bb0ca452e7d7723ccac9a810)将有效地执行* new_image = a * image + beta *。 不过，我们想告诉你如何访问每个像素。 在任何情况下，这两种方法给出了相同的结果，但convertTo更优化，工作速度更快。

#### Result

* Running our code and using α=2.2 β=50
```C++
    $ ./BasicLinearTransforms lena.jpg
    Basic Linear Transforms
    -------------------------
    * Enter the alpha value [1.0-3.0]: 2.2
    * Enter the beta value [0-100]: 50
```
* We get this:

![结果](https://docs.opencv.org/3.1.0/Basic_Linear_Transform_Tutorial_Result_big.jpg)


