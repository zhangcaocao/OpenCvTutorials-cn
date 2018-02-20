---
title: 【OpenCvTutorials3.1中文翻译】- 操作图像
date:  2018-02-05 14:07:34
tags: [OpenCvTutorials3.1中文翻译]
categories: [原创]
thumbnail:  [http://otneosm59.bkt.clouddn.com/18_2_5_02.png]
toc: true
---
<!-- Go to www.addthis.com/dashboard to customize your tools -->
<script type="text/javascript" src="//s7.addthis.com/js/300/addthis_widget.js#pubid=ra-59cb1e1aae525c27"></script>
### [Operations with images](https://docs.opencv.org/3.1.0/d5/d98/tutorial_mat_operations.html)

#### 输入输出

##### 图片

从文件加载图像：
```C++
Mat img = imread(filename)
```
如果您读取一个jpg文件，默认情况下会创建一个3通道图像。 如果您需要灰度图像，请使用：
```C++
Mat img = imread(filename, 0);
```
<!-- more -->
> 注意：
文件格式由其内容决定（前几个字节）将图像保存到文件中：

```C++
imwrite(filename, img);
```
> 注意： 该文件的格式是由其扩展名决定的。
使用imdecode和imencode是从内存读/写图像而不是文件。


### 图像的基本操作
#### 访问像素强度值
为了获得像素强度值，您必须知道图像的类型和通道的数量。 以下是单通道灰度图像（类型8UC1）和像素坐标x和y的示例：
```C++
Scalar intensity = img.at<uchar>(y, x);
```
intensity.val [0]包含从0到255的值。请注意x和y的排序。因为在OpenCV中，图像由与矩阵相同的结构表示，所以我们在两种情况下都使用相同的约定 - 基于0的行索引（或y坐标）先出现，然后是基于0的列索引（或x坐标)。 另外，您可以使用以下表示法：
```C++
Scalar intensity = img.at<uchar>(Point(x, y));
```
现在让我们考虑一个BGR颜色排序的3通道图像（imread返回的默认格式）：
```C++
Vec3b intensity = img.at<Vec3b>(y, x);
uchar blue = intensity.val[0];
uchar green = intensity.val[1];
uchar red = intensity.val[2];
```
您可以对浮点图像使用相同的方法（例如，可以通过在3通道图像上运行Sobel来获得这样的图像）：

```C++
Vec3f intensity = img.at<Vec3f>(y, x);
float blue = intensity.val[0];
float green = intensity.val[1];
float red = intensity.val[2];
```
可以使用相同的方法来改变像素强度：
```C++
img.at<uchar>(y, x) = 128;
```
在OpenCV中有函数，特别是来自calib3d模块的函数，比如projectPoints，它以Mat的形式获取一个2D或3D点的数组。 矩阵应该只包含一列，每一行对应一个点，矩阵类型应相应为32FC2或32FC3。 这样的矩阵可以很容易地从`std::vector`构造出来：
```C++
vector<Point2f> points;
//... fill the array填充数组
Mat pointsMat = Mat(points);
```
可以使用相同的方法`Mat::at`在这个矩阵中访问一个点：
```C++
Point2f point = pointsMat.at<Point2f>(i, 0);
```
##### 内存管理和引用计数
Mat是一种保持矩阵/图像特征（行和列号，数据类型等）和指向数据的指针的结构。 所以没有任何东西阻止我们有几个Mat实例对应于相同的数据。 一个Mat保留一个引用计数，告诉数据是否在Mat的特定实例被销毁时被释放。 以下是创建两个矩阵而不复制数据的示例：
> 译者注：
这里的计数参考：[Mat-基本的图像容器](http://little-rocket.cn/2018/02/04/%E3%80%90OpenCvTutorials3-1%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91%E3%80%91-%20Mat-%E5%9F%BA%E6%9C%AC%E7%9A%84%E5%9B%BE%E5%83%8F%E5%AE%B9%E5%99%A8/)

```C++
std::vector<Point3f> points;
// .. fill the array
Mat pointsMat = Mat(points).reshape(1);
```
最后，我们得到了3列32FC1矩阵，而不是1列32FC3矩阵。 pointsMat使用来自点的数据，并且在销毁时不会释放内存。 然而，在这个特定的例子中，开发者必须确保points的寿命比pointsMat的长。 如果我们需要复制数据，则使用例如[cv::Mat::copyTo](https://docs.opencv.org/3.1.0/d3/d63/classcv_1_1Mat.html#a4331fa88593a9a9c14c0998574695ebb)或[cv::Mat::clone](https://docs.opencv.org/3.1.0/d3/d63/classcv_1_1Mat.html#afb01ff6b2231b72f55618bfb66a5326b)的方法：

```C++
Mat img = imread("image.jpg");
Mat img1 = img.clone();
```
相反，如果C API需要由开发人员创建输出图像，则可以为每个函数提供一个空的Mat的输出。每一次使用`Mat :: create`实现目标矩阵。 如果该方法为空，则该方法为矩阵分配数据。 如果它不是空的并且具有正确的大小和类型，则该方法不做任何事情。 但是，如果大小或类型与输入参数不同，则会释放（并丢失）数据，并分配新数据。 例如：
```C++
Mat img = imread("image.jpg");
Mat sobelx;
Sobel(img, sobelx, CV_32F, 1, 0);
```
##### 原始的操作
在矩阵上定义了许多方便的操作符。 例如，下面是我们如何从现有的灰度图像“img”中制作黑色图像：
```C++
img = Scalar(0);
```
选择一个感兴趣的区域(ROI)：
```C++
Rect r(10, 10, 100, 100);
Mat smallImg = img(r);
```
从Mat到C API数据结构的转换：
```C++
Mat img = imread("image.jpg");
IplImage img1 = img;
CvMat m = img;
```
请注意，这里没有数据复制。

从彩色转换到灰度：
```C++
Mat img = imread("image.jpg"); // loading a 8UC3 image
Mat grey;
cvtColor(img, grey, COLOR_BGR2GRAY);
```

将图像类型从8UC1更改为32FC1：

```C++
src.convertTo(dst, CV_32F);
```
##### 可视化图像
在开发过程中查看算法的中间结果是非常有用的。 OpenCV提供了一种可视化图像的便捷方式。 一个8U图像可以显示使用：

```C++
Mat img = imread("image.jpg");
namedWindow("image", WINDOW_AUTOSIZE);
imshow("image", img);
waitKey();
```
对[waitKey（）](https://docs.opencv.org/3.1.0/d7/dfc/group__highgui.html#ga5628525ad33f52eab17feebcfba38bd7)的调用会启动一个消息传递循环，等待“图像”窗口中的关键笔画。 32F图像需要转换为8U类型。 例如：

```C++
Mat img = imread("image.jpg");
Mat grey;
cvtColor(img, grey, COLOR_BGR2GRAY);
Mat sobelx;
Sobel(grey, sobelx, CV_32F, 1, 0);
double minVal, maxVal;
minMaxLoc(sobelx, &minVal, &maxVal); //find minimum and maximum intensities
Mat draw;
sobelx.convertTo(draw, CV_8U, 255.0/(maxVal - minVal), -minVal * 255.0/(maxVal - minVal));
namedWindow("image", WINDOW_AUTOSIZE);
imshow("image", draw);
waitKey();
```
####  译者注：
函数参考链接：
* [cvtColor](https://docs.opencv.org/3.1.0/d7/d1b/group__imgproc__misc.html#ga397ae87e1288a81d2363b61574eb8cab)
* [Sobel](https://docs.opencv.org/3.1.0/d4/d86/group__imgproc__filter.html#gacea54f142e81b6758cb6f375ce782c8d)
* [minMaxLoc](https://docs.opencv.org/3.1.0/d2/de8/group__core__array.html#gab473bf2eb6d14ff97e89b355dac20707)
* [waitkey](https://docs.opencv.org/3.1.0/d7/dfc/group__highgui.html#ga5628525ad33f52eab17feebcfba38bd7)