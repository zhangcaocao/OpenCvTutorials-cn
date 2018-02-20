---
title: 【OpenCvTutorials3.1中文翻译】-[Mat-基本的图像容器]
date: 2018-02-04 14:16:25
tags: [OpenCvTutorials3.1中文翻译]
categories: [原创]
thumbnail:  https://docs.opencv.org/3.1.0/MatBasicImageForComputer.jpg
toc: true
---
<!-- Go to www.addthis.com/dashboard to customize your tools -->
<script type="text/javascript" src="//s7.addthis.com/js/300/addthis_widget.js#pubid=ra-59cb1e1aae525c27"></script>
### [Mat-基本的图像容器](https://docs.opencv.org/3.1.0/d6/d6d/tutorial_mat_the_basic_image_container.html)
#### Goal

我们有许多方法从现实世界中获取数字图像：数码相机、扫描仪、计算机断层扫描和核磁共振成像等。在任何情况下，我们（人类）看到的都是图像，但是将其转换为数字设备时，我们记录的便是图像中每一点的数字化的值。

![数字设备的图像](https://docs.opencv.org/3.1.0/MatBasicImageForComputer.jpg)
例如，在上面的图片中，您可以看到汽车只不过表现为一个包含像素点所有强度值的矩阵。我们如何获取和存储像素值可能会根据我们的需要而有所不同，但最终计算机世界中的所有图像可能会被简化为描述矩阵本身的数字矩阵和其他信息。OpenCV是一个计算机视觉库，其重点是处理和操作这些信息。 因此，您首先需要熟悉的是OpenCV如何存储和处理图像。

<!-- more -->
#### Mat
 自从2001年以来，OpenCV就已经出现了。那时候，这个库是围绕一个C接口构建 的，并且将图像存储在内存中，他们使用了一个被称为[IplImage](https://docs.opencv.org/3.1.0/d6/d5b/structIplImage.html)的C结构体。 在大多数较老的教程和教材中，您将看到这一点。这个问题就是把C语言的所有缺点都展现了出来。最大的问题是手动内存管理。它建立在假定用户负责照顾内存分配和取消分配。虽然这对于较小的程序来说并不是问题，但是一旦你的代码库增长，处理所有这些将更加困难，而不是专注于解决你的开发目标。

 幸运的是，C ++出现了，通过自动内存管理（或多或少）引入了类的概念，使用户更容易。 好消息是C ++与C完全兼容，所以不会出现兼容性问题。 因此，OpenCV 2.0引入了一个新的C ++接口，它提供了一种新的处理方式，这意味着你不需要进行内存管理，使你的代码简洁（少写，实现更多）。 C ++接口的主要缺点是目前许多嵌入式开发系统只支持C.因此，除非你是针对嵌入式平台，否则使用旧的方法是没有意义的（除非你是个受虐狂的程序员，否则你就是自找麻烦）。

关于Mat你首先需要了解，你不再需要手动分配内存，和在不需要的时候释放内存。虽然这样做仍然是可能的，但大部分OpenCV函数都会自动分配输出数据。一个额外的好处是，如果你传递一个已经存在的Mat对象（这个对象已经为矩阵分配了所需的空间），并且它将被重用。

Mat基本上是一个包含两个数据部分的类：矩阵头部（包含矩阵大小，用于存储的方法，存储矩阵的地址等信息）以及包含的像素值（取决于所选择的存储方法取任何维度）。 矩阵头的大小是恒定的，但是矩阵本身的大小可能会随着图像的不同而变化，并且通常会大几个数量级。

OpenCV是一个图像处理库。 它包含大量的图像处理功能。 为了解决计算上的挑战，大多数时候你最终会使用库的多个函数。 因此，将图像传递给函数是一种常见的做法。 我们不应该忘记，我们正在谈论的图像处理算法，往往是相当计算量。 我们最不愿做的就是进一步降低程序的速度，使程序中潜在不必要的大图像的副本。

为了解决这个问题，OpenCV使用了一个引用计数系统。这个想法是，每个Mat对象都有它自己的头，但是矩阵可以通过使它们的矩阵指针指向相同的地址而在它们的两个实例之间共享。 而且，复制操作符`只会将头部和指针复制到大矩阵`，而不是数据本身。

```C++
Mat A, C;                          // creates just the header parts
A = imread(argv[1], IMREAD_COLOR); // here we'll know the method used (allocate matrix)
Mat B(A);                                 // Use the copy constructor
C = A;                                    // Assignment operator
```
上述所有的对象最后指向同一个单一的数据矩阵。 然而，他们的headers是不同的，使用它们中的任何一个都会影响其他所有的headers。 实际上，不同的对象只是为相同的基础数据提供不同的访问方法。 不过，他们的headers是不同的。 真正有趣的部分是，你可以创建headers，只引用全部数据的一个小节。 例如，要在图像中创建感兴趣区域（ROI），只需使用新边界创建一个新的headers：

```C++
Mat D (A, Rect(10, 10, 100, 100) ); // using a rectangle
Mat E = A(Range::all(), Range(1,3)); // using row and column boundaries
```

现在您可能会问，矩阵本身是否属于多个Mat对象，在不再需要的时候，Mat对象负责清理它。 关于这个问题的简短的回答是：使用它的最后一个对象。 这是通过使用引用计数机制来处理的。 每当有人复制一个Mat对象的头部时，矩阵的计数器就增加了。 每当头部被清除时，这个计数器就会减少。 当计数器达到零时，矩阵也被释放。 有时你也想复制矩阵本身，所以OpenCV提供了[cv::Mat::clone（）](https://docs.opencv.org/3.1.0/d3/d63/classcv_1_1Mat.html#afb01ff6b2231b72f55618bfb66a5326b)和[cv::Mat::copyTo（）](https://docs.opencv.org/3.1.0/d3/d63/classcv_1_1Mat.html#a4331fa88593a9a9c14c0998574695ebb)函数。

```C++
Mat F = A.clone();
Mat G;
A.copyTo(G);
```
现在修改F或G不会影响Mat头指向的矩阵。 你需要记住的是：

* OpenCV函数的输出图像分配是自动的（除非另有规定）。
* 您不需要考虑使用OpenCV的C ++接口进行内存管理。
* 赋值运算符和复制构造函数只复制标题。
* 图像的底层矩阵可以使用[cv::Mat::clone（）](https://docs.opencv.org/3.1.0/d3/d63/classcv_1_1Mat.html#afb01ff6b2231b72f55618bfb66a5326b)和[cv::Mat::copyTo（）](https://docs.opencv.org/3.1.0/d3/d63/classcv_1_1Mat.html#a4331fa88593a9a9c14c0998574695ebb)函数来复制。

#### 存储方法
这是关于如何存储像素值。 您可以选择使用的颜色空间和数据类型。 色彩空间是指我们如何组合颜色组件以编码给定的颜色。 最简单的就是我们可以使用的颜色是黑色和白色的灰度。 这些组合让我们创造出许多灰色梯度。

对于彩色的储存方式，我们有更多的方法供选择。对于丰富多彩的方式，我们有更多的方法可供选择。 他们每个分解成三个或四个基本组件，我们可以使用这些组合来创建其他。 最流行的是RGB，主要是因为这也是我们的眼睛如何建立颜色。 它的基本颜色是红色，绿色和蓝色。 为了编码颜色的透明度，有时需要添加第四个元素：alpha（A）。

然而，还有许多其他颜色系统各有其优点：

* RGB是最常见的，因为我们的眼睛使用类似的东西，但请记住，OpenCV标准显示系统使用BGR色彩空间（红色和蓝色通道的开关）组合颜色。
* HSV和HLS将颜色分解为色调，饱和度和亮度/亮度分量，这是我们描述颜色的更自然的方式。 例如，您可能会忽略最后一个组件，从而使您的算法对输入图像的光照条件不那么敏感。
* YCrCb被流行的JPEG图像格式所使用。
* CIE L* a * b* 是一个感知上均匀的颜色空间，如果您需要测量给定颜色与另一种颜色之间的距离，则该颜色空间非常方便。

每一个组成的组建都有他们的有效域， 这决定了使用的数据类型。我们如何存储组件定义了我们对其域的控制权。 可能的最小数据类型是char，也就是一个字节或8位。这可能是无符号的（所以可以存储从0到255的值）或有符号的（从-127到+127的值）。尽管在三个组成部分的情况下，这已经提供了1600万个可能的颜色来表示（比如在RGB的情况下），我们可以通过使用float（4字节= 32位）或者double（8字节= 64位）数据来更精细的控制每个组成部分。不过，请记住，增加组件的大小也会增加内存中整个图片的大小。

#### 显式创建一个Mat对象

在[Load, Modify, and Save an Image](https://docs.opencv.org/3.1.0/db/d64/tutorial_load_save_image.html)教程中，您已经学习了如何使用cv :: imwrite（）函数将矩阵写入图像文件。 但是，出于调试目的，查看实际值要方便得多。 您可以使用Mat的<<运算符来执行此操作。 请注意，这只适用于二维矩阵。

尽管Mat作为一个图像容器非常适合，它也是一个通用的矩阵类。 因此，可以创建和操作多维矩阵。 您可以通过多种方式创建一个Mat对象：

* [cv::Mat::Mat ](https://docs.opencv.org/3.1.0/d3/d63/classcv_1_1Mat.html#af1d014cecd1510cdf580bf2ed7e5aafc)构造函数：
```C++
    Mat M(2,2, CV_8UC3, Scalar(0,0,255));
    cout << "M = " << endl << " " << M << endl << endl;
```
![运行结果](https://docs.opencv.org/3.1.0/MatBasicContainerOut1.png)

对于二维和多通道图像，我们首先定义它们的大小：明智的行和列的数目。

然后，我们需要指定用于存储元素的数据类型和每个矩阵点的通道数量。 要做到这一点，我们有多个按照以下惯例构建的定义：

```C++
CV_[The number of bits per item][Signed or Unsigned][Type Prefix]C[The channel number]
```

例如，CV_8UC3意味着我们使用8位长的无符号字符类型，每个像素有三个这样的字符串来形成三个通道。 这是预定义的最多四个通道数目。[cv :: Scalar](https://docs.opencv.org/3.1.0/dc/d84/group__core__basic.html#ga599fe92e910c027be274233eccad7beb)是四元素的short矢量。 指定这个，你可以用一个自定义的值初始化所有的矩阵点。 如果您需要更多，您可以使用上面的宏创建类型，如下所示在括号中设置通道号。

* 使用C / C ++数组并通过构造函数初始化

```c++
    int sz[3] = {2,2,2};
    Mat L(3,sz, CV_8UC(1), Scalar::all(0));
```
上面的例子展示了如何创建一个具有两个以上维度的矩阵。 指定其维度，然后传递包含每个维度大小的指针，其余维持不变。

* [cv :: Mat :: create](https://docs.opencv.org/3.1.0/d3/d63/classcv_1_1Mat.html#a55ced2c8d844d683ea9a725c60037ad0)函数：

```c++
    M.create(4,4, CV_8UC(2));
    cout << "M = "<< endl << " "  << M << endl << endl;
```
![运行结果](https://docs.opencv.org/3.1.0/MatBasicContainerOut2.png)

这种结构不能初始化矩阵值。 如果新的尺寸不适合旧的，它将只重新分配它的矩阵数据存储器。

MATLAB风格初始值设定项：[cv::Mat::zeros](https://docs.opencv.org/3.1.0/d3/d63/classcv_1_1Mat.html#a0b57b6a326c8876d944d188a46e0f556)，[cv::Mat::ones](https://docs.opencv.org/3.1.0/d3/d63/classcv_1_1Mat.html#a69ae0402d116fc9c71908d8508dc2f09)，[cv::Mat::eye](https://docs.opencv.org/3.1.0/d3/d63/classcv_1_1Mat.html#a2cf9b9acde7a9852542bbc20ef851ed2)。 指定要使用的大小和数据类型：
```C++
    Mat E = Mat::eye(4, 4, CV_64F);
    cout << "E = " << endl << " " << E << endl << endl;
    Mat O = Mat::ones(2, 2, CV_32F);
    cout << "O = " << endl << " " << O << endl << endl;
    Mat Z = Mat::zeros(3,3, CV_8UC1);
    cout << "Z = " << endl << " " << Z << endl << endl;
```
![运行结果](https://docs.opencv.org/3.1.0/MatBasicContainerOut3.png)

* 对于小型矩阵，您可以使用逗号分隔的初始值设定项：

```C++
    Mat C = (Mat_<double>(3,3) << 0, -1, 0, -1, 5, -1, 0, -1, 0);
    cout << "C = " << endl << " " << C << endl << endl;
```
![运行结果](https://docs.opencv.org/3.1.0/MatBasicContainerOut6.png)

* 为现有的Mat对象和[cv::Mat::clone](https://docs.opencv.org/3.1.0/d3/d63/classcv_1_1Mat.html#afb01ff6b2231b72f55618bfb66a5326b)或[cv::Mat::copyTo](https://docs.opencv.org/3.1.0/d3/d63/classcv_1_1Mat.html#a4331fa88593a9a9c14c0998574695ebb)创建一个新的头部。

```C++
    Mat RowClone = C.row(1).clone();
    cout << "RowClone = " << endl << " " << RowClone << endl << endl;
```
![运行结果](https://docs.opencv.org/3.1.0/MatBasicContainerOut7.png)


> Note:
> 你可以使用 [cv :: randu（）](https://docs.opencv.org/3.1.0/d2/de8/group__core__array.html#ga1ba1026dca0807b27057ba6a49d258c0)函数填充随机值的矩阵。 您需要给出随机值的下限和上限值：
```C++
    Mat R = Mat(3, 2, CV_8UC3);
    randu(R, Scalar::all(0), Scalar::all(255));
```

#### 输出格式
在上面的例子中，你可以看到默认的格式化选项。 然而，OpenCV允许你格式化你的矩阵输出：

* Default:

```C++
    cout << "R (default) = " << endl <<        R           << endl << endl;
```
![运行结果](https://docs.opencv.org/3.1.0/MatBasicContainerOut8.png)

* Python:

```python
    cout << "R (python)  = " << endl << format(R, Formatter::FMT_PYTHON) << endl << endl;
```
![运行结果](https://docs.opencv.org/3.1.0/MatBasicContainerOut16.png)

* Comma separated values (CSV):
```C++
    cout << "R (csv)     = " << endl << format(R, Formatter::FMT_CSV   ) << endl << endl;
```
![运行结果](https://docs.opencv.org/3.1.0/MatBasicContainerOut10.png)


* Numpy:

```C++
cout << "R (c)       = " << endl << format(R, Formatter::FMT_C     ) << endl << endl;
```
![Numpy](https://docs.opencv.org/3.1.0/MatBasicContainerOut11.png)


#### 其他常见项目的输出

OpenCV也支持通过<<运算符来输出其他常见的OpenCV数据结构：

* 2D Point:

```C++
    Point2f P(5, 1);
    cout << "Point (2D) = " << P << endl << endl;
```
![2D Point](https://docs.opencv.org/3.1.0/MatBasicContainerOut12.png)

* 3D Point:

```C++
    Point3f P3f(2, 6, 7);
    cout << "Point (3D) = " << P3f << endl << endl;
```
![3D Point](https://docs.opencv.org/3.1.0/MatBasicContainerOut13.png)

* std::vector via [cv::Mat](https://docs.opencv.org/3.1.0/d3/d63/classcv_1_1Mat.html)

```C++
    vector<float> v;
    v.push_back( (float)CV_PI);   v.push_back(2);    v.push_back(3.01f);
    cout << "Vector of floats via Mat = " << Mat(v) << endl << endl;
```
![输出](https://docs.opencv.org/3.1.0/MatBasicContainerOut14.png)

* std::vector of points：

```C++
    vector<Point2f> vPoints(20);
    for (size_t i = 0; i < vPoints.size(); ++i)
        vPoints[i] = Point2f((float)(i * 5), (float)(i % 7));
    cout << "A vector of 2D Points = " << vPoints << endl << endl;
```
![输出](https://docs.opencv.org/3.1.0/MatBasicContainerOut15.png)

这里的大部分示例都包含在一个小的控制台应用程序中。 您可以从[这里](https://github.com/Itseez/opencv/tree/master/samples/cpp/tutorial_code/core/mat_the_basic_image_container/mat_the_basic_image_container.cpp)或cpp示例的核心部分下载它。

你也可以在YouTube上找到这个快速的视频演示。

<iframe width="560" height="349" src="https://www.youtube.com/embed/1tibU7vGWpk" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
