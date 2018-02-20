---
title: 【OpenCvTutorials3.1中文翻译】- 离散傅里叶变换
toc: true
thumbnail: 'http://image.little-rocket.cn/18_02_16_15.jpg'
date: 2018-02-16 16:10:10
tags: [OpenCvTutorials3.1中文翻译]
categories: [原创]
---
### [Discrete Fourier Transform](https://docs.opencv.org/3.1.0/d8/d01/tutorial_discrete_fourier_transform.html)

<!-- Go to www.addthis.com/dashboard to customize your tools -->
<script type="text/javascript" src="//s7.addthis.com/js/300/addthis_widget.js#pubid=ra-59cb1e1aae525c27"></script>

#### Goal:
我们将为以下问题寻求答案：
* 什么是傅立叶变换以及为什么使用它？
* 如何在OpenCV中做到这一点？
* 函数的用法如：[cv::copyMakeBorder（）](https://docs.opencv.org/3.1.0/d2/de8/group__core__array.html#ga2ac1049c2c3dd25c2b41bffe17658a36)，[cv::merge（）](https://docs.opencv.org/3.1.0/d2/de8/group__core__array.html#ga7d7b4d6c6ee504b30a20b1680029c7b4)，[cv::dft（）](https://docs.opencv.org/3.1.0/d2/de8/group__core__array.html#gadd6cf9baf2b8b704a11b5f04aaf4f39d)，[cv :: getOptimalDFTSize（）](https://docs.opencv.org/3.1.0/d2/de8/group__core__array.html#ga6577a2e59968936ae02eb2edde5de299)，[cv :: log（）](https://docs.opencv.org/3.1.0/d2/de8/group__core__array.html#ga937ecdce4679a77168730830a955bea7)和 [cv :: normalize（）](https://docs.opencv.org/3.1.0/dc/d84/group__core__basic.html#ga1b6a396a456c8b6c6e4afd8591560d80)。

<!-- more -->

#### Source code
你可以在[这里](https://github.com/Itseez/opencv/tree/master/samples/cpp/tutorial_code/core/discrete_fourier_transform/discrete_fourier_transform.cpp)或者是在`samples/cpp/tutorial_code/core/discrete_fourier_transform/discrete_fourier_transform.cpp`获取代码。

以下是[cv::dft（）](https://docs.opencv.org/3.1.0/d2/de8/group__core__array.html#gadd6cf9baf2b8b704a11b5f04aaf4f39d)的一个示例用法：

```C++
#include "opencv2/core/core.hpp"
#include "opencv2/imgproc/imgproc.hpp"
#include "opencv2/imgcodecs.hpp"
#include "opencv2/highgui/highgui.hpp"

#include <iostream>

using namespace cv;
using namespace std;

static void help(char* progName)
{
    cout << endl
        <<  "This program demonstrated the use of the discrete Fourier transform (DFT). " << endl
        <<  "The dft of an image is taken and it's power spectrum is displayed."          << endl
        <<  "Usage:"                                                                      << endl
        << progName << " [image_name -- default ../data/lena.jpg] "               << endl << endl;
}

int main(int argc, char ** argv)
{
    help(argv[0]);

    const char* filename = argc >=2 ? argv[1] : "../data/lena.jpg";

    Mat I = imread(filename, IMREAD_GRAYSCALE);
    if( I.empty())
        return -1;

    Mat padded;                            //expand input image to optimal size
    int m = getOptimalDFTSize( I.rows );
    int n = getOptimalDFTSize( I.cols ); // on the border add zero values
    copyMakeBorder(I, padded, 0, m - I.rows, 0, n - I.cols, BORDER_CONSTANT, Scalar::all(0));

    Mat planes[] = {Mat_<float>(padded), Mat::zeros(padded.size(), CV_32F)};
    Mat complexI;
    merge(planes, 2, complexI);         // Add to the expanded another plane with zeros

    dft(complexI, complexI);            // this way the result may fit in the source matrix

    // compute the magnitude and switch to logarithmic scale
    // => log(1 + sqrt(Re(DFT(I))^2 + Im(DFT(I))^2))
    split(complexI, planes);                   // planes[0] = Re(DFT(I), planes[1] = Im(DFT(I))
    magnitude(planes[0], planes[1], planes[0]);// planes[0] = magnitude
    Mat magI = planes[0];

    magI += Scalar::all(1);                    // switch to logarithmic scale
    log(magI, magI);

    // crop the spectrum, if it has an odd number of rows or columns
    magI = magI(Rect(0, 0, magI.cols & -2, magI.rows & -2));

    // rearrange the quadrants of Fourier image  so that the origin is at the image center
    int cx = magI.cols/2;
    int cy = magI.rows/2;

    Mat q0(magI, Rect(0, 0, cx, cy));   // Top-Left - Create a ROI per quadrant
    Mat q1(magI, Rect(cx, 0, cx, cy));  // Top-Right
    Mat q2(magI, Rect(0, cy, cx, cy));  // Bottom-Left
    Mat q3(magI, Rect(cx, cy, cx, cy)); // Bottom-Right

    Mat tmp;                           // swap quadrants (Top-Left with Bottom-Right)
    q0.copyTo(tmp);
    q3.copyTo(q0);
    tmp.copyTo(q3);

    q1.copyTo(tmp);                    // swap quadrant (Top-Right with Bottom-Left)
    q2.copyTo(q1);
    tmp.copyTo(q2);

    normalize(magI, magI, 0, 1, NORM_MINMAX); // Transform the matrix with float values into a
                                            // viewable image form (float between values 0 and 1).

    imshow("Input Image"       , I   );    // Show the result
    imshow("spectrum magnitude", magI);
    waitKey();

    return 0;
}
```

#### Explanation:

傅里叶变换将图像分解为其正弦和余弦分量.也就是说，它会将图像从其空间域转换到其频域。 这个思想是任何函数都可以用无限的正弦函数和余弦函数的和来近似，傅立叶变换是一种如何做到这一点的方法。 在数学上二维图像傅里叶变换是：
![公式1](http://image.little-rocket.cn/18_02_16_10.PNG)

这里f是空间域的图像值，F是频域的图像值。转换的结果是复数。通过`a real image and a complex image `或 通过幅度和相位图像来显示都是可能的。然而，在整个图像处理算法中，只有幅度图像是有趣的，因为这包含了我们需要的关于图像几何结构的所有信息。不过，你如果打算对这些形式的图像作一些修改，你就需要重新转换它，并且保留着两个 image。

在这个例子中，我将展示如何计算和显示傅里叶变换的幅度图像。数字图像是离散的。这意味着他们可能会从一个给定的域值中获得一个值。例如，在基本灰度级中，图像值通常在0到255之间，因此，傅里叶变换也需要是离散傅立叶变换（DFT）。每当需要从几何角度确定图像的结构时，您都会想要使用它。以下是要遵循的步骤（如果是灰度输入图像I）：

###### 1、**将图像展开至最佳尺寸。** 
DFT的性能取决于图像大小。它往往是最快速度下的图像尺寸，它是数字2,3和5的倍数。因此，为了达到最佳性能，可以将边界值填充到获得具有这种特征的大小。[cv::getOptimalDFTSize（）](https://docs.opencv.org/3.1.0/d2/de8/group__core__array.html#ga6577a2e59968936ae02eb2edde5de299) 返回这个最佳尺寸，我们可以使用 [cv::copyMakeBorder()](https://docs.opencv.org/3.1.0/d2/de8/group__core__array.html#ga2ac1049c2c3dd25c2b41bffe17658a36)函数来扩展图像的边界：

```C++
Mat padded;                            //expand input image to optimal size
int m = getOptimalDFTSize( I.rows );
int n = getOptimalDFTSize( I.cols ); // on the border add zero pixels
copyMakeBorder(I, padded, 0, m - I.rows, 0, n - I.cols, BORDER_CONSTANT, Scalar::all(0));
```
附加像素初始化为零。

###### 2、**为实部和虚部做好准备。**  
傅立叶变换的结是复数。这意味着对于每个图像值，结果是两个图像值（每个组件一个）。而且，频域范围远大于其空间对应域。因此，我们通常至少以浮点格式存储这些数据。因此，我们会将我们的输入图像转换为此类型，并用另一个通道来扩展它以保存虚部( `complex values` )：
```C++
Mat planes[] = {Mat_<float>(padded), Mat::zeros(padded.size(), CV_32F)};
Mat complexI;
merge(planes, 2, complexI);         // Add to the expanded another plane with zeros
```

###### 3、**进行离散傅里叶变换。** 
可以进行就地计算（与输出相同的输入）：
```C++
dft(complexI, complexI);            // this way the result may fit in the source matrix
```

###### 4、**将实部和虚部的值转换为数值。** 
一个复数有一个实数（Re）和一个虚数（ Im）部分。 DFT的结果是复数。
DFT的大小是：
![公式2](http://image.little-rocket.cn/18_02_16_11.PNG)

用OpenCv代码实现：
```C++
split(complexI, planes);                   // planes[0] = Re(DFT(I), planes[1] = Im(DFT(I))
magnitude(planes[0], planes[1], planes[0]);// planes[0] = magnitude
Mat magI = planes[0];
```

###### 5、**切换到对数比例。 （switch to logarithmic scale）**
事实证明，傅里叶系数的动态范围太大而不能显示在屏幕上。我们有一些很小和一些很高的变化值，我们不能这样观察。
因此，高值将全部变成白点，而小值变成黑色。 要使用灰度值来进行可视化，我们可以将线性比例转换为对数：
![公式3](http://image.little-rocket.cn/18_02_16_12.PNG)

用OpenCv代码实现：
```C++
magI += Scalar::all(1);     // switch to logarithmic scale
log(magI, magI);
```

###### 6、** 裁切和重新排列。** 
请记住，在第一步，我们扩大了图像。那么，是时候抛弃新引入的值了。为了可视化，我们还需要重新排列结果的象限，以便原点（0，0）与图像中心相对应。

```C++
magI = magI(Rect(0, 0, magI.cols & -2, magI.rows & -2));
int cx = magI.cols/2;
int cy = magI.rows/2;
Mat q0(magI, Rect(0, 0, cx, cy));   // Top-Left - Create a ROI per quadrant
Mat q1(magI, Rect(cx, 0, cx, cy));  // Top-Right
Mat q2(magI, Rect(0, cy, cx, cy));  // Bottom-Left
Mat q3(magI, Rect(cx, cy, cx, cy)); // Bottom-Right
Mat tmp;                           // swap quadrants (Top-Left with Bottom-Right)
q0.copyTo(tmp);
q3.copyTo(q0);
tmp.copyTo(q3);
q1.copyTo(tmp);                    // swap quadrant (Top-Right with Bottom-Left)
q2.copyTo(q1);
tmp.copyTo(q2);
```

###### 7、**规范化。** 
这是为了可视化目的而再次完成的。我们现在有这些幅度，但是这仍然超出了我们的图像显示范围（0到1）。我们使用[cv::normalize（）](https://docs.opencv.org/3.1.0/dc/d84/group__core__basic.html#ga1b6a396a456c8b6c6e4afd8591560d80)函数将我们的值规范化到这个范围。

```C++
normalize(magI, magI, 0, 1, NORM_MINMAX); // Transform the matrix with float values into a
                                          // viewable image form (float between values 0 and 1).
```

#### Result：
关于应用是确定图像中存在的几何取向。例如，让我们看看文本是否是水平的？`Looking at some text you'll notice that the text lines sort of form also horizontal lines and the letters form sort of vertical lines. `  在傅里叶变换的情况下，也可以看到文本片段的这两个主要组成部分。
让我们使用这个[水平](https://github.com/opencv/opencv/blob/master/samples/data/imageTextN.png)和[旋转](https://github.com/opencv/opencv/blob/master/samples/data/imageTextR.png)文本的图像。

![horizontal text](http://image.little-rocket.cn/18_02_16_14.PNG)
![rotated text](http://image.little-rocket.cn/18_02_16_13.PNG)

###### In case of the horizontal text（水平）:
![结果1](http://image.little-rocket.cn/18_02_16_15.jpg)

###### In case of a rotated text:
![结果2](http://image.little-rocket.cn/18_02_16_16.jpg)

您可以看到，频域中最有影响的分量（幅度图像上最亮的点）遵循图像上物体的几何旋转。 由此我们可以计算偏移量并执行图像旋转以校正最终的对准。
