---
title: 【OpenCvTutorials3.1中文翻译】- 如何使用OpenCV扫描图像，查找表和时间测量
date: 2018-02-04 15:25:09
tags: [OpenCvTutorials3.1中文翻译]
categories: [原创]
thumbnail:  https://docs.opencv.org/3.1.0/tutorial_how_matrix_stored_2.png
toc: true
---
<!-- Go to www.addthis.com/dashboard to customize your tools -->
<script type="text/javascript" src="//s7.addthis.com/js/300/addthis_widget.js#pubid=ra-59cb1e1aae525c27"></script>

### [How to scan images, lookup tables and time measurement with OpenCV](https://docs.opencv.org/3.1.0/db/da5/tutorial_how_to_scan_images.html)

#### 目标

我们将为以下问题寻求答案：

* 如何通过图像的每个像素？
* OpenCV矩阵值如何存储？
* 如何衡量我们算法的性能？
* 什么是查找表，为什么使用它们？


<!-- more -->
#### 我们的测试用例
让我们考虑一个简单的色彩还原方法。. 通过使用无符号字符C和C++矩阵项目存储，一个像素的渠道可能有多达256个不同的值。对于三通道图像这可能允许太多颜色的形成(确切的说是1600万)。工作有这么多颜色的色调可以给我们的算法性能的一个沉重的打击。但是，有时只用更少的工作就可以得到相同的最终结果。

在这种情况下，我们通常会减少色彩空间。 这意味着我们将色彩空间当前值与一个新的输入值分开，以较少的颜色结束。 例如，0与9之间的每个值都取新值0，10至19之间的每一个值10等等。

当你用一个int值分割一个uchar（unsigned char - 又名0到255之间的值）值时，结果也是char。 这些值可能只是char值。 因此，任何分数都会被舍入。 利用这个事实，uchar域中的上面的操作可以表示为：

<script type="text/javascript" async
  src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.2/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

<p>
  $$In= (\frac {I_{o}}{10}) * 10.$$
</p>

> In: I_new; Io: I_old;

一个简单的色彩空间减少算法将包括只是通过一个图像矩阵的每个像素应用这个公式。 值得注意的是，我们做了分裂和乘法运算。 这些操作对于一个系统来说是非常昂贵的。 如果可能的话，通过使用更便宜的操作（例如几次减法，加法或最好的情况下简单的分配）来避免它们是值得的。 此外，请注意，对于上层操作，我们只有有限的输入值。 在uchar系统的情况下，这是256。

因此，对于较大的图像，事先计算所有可能的值并在分配期间通过使用查找表来进行分配将是明智的。 查找表是简单的数组（具有一个或多个维度），对于给定的输入值变化保存最终的输出值。 它的优势在于我们不需要计算，我们只需要读取结果。

我们的测试用例程序（和这里提供的示例）将执行以下操作：读取控制台行参数图像（可能是彩色或灰度级 - 控制台行参数），并将缩减应用于给定的控制台行参数整数值。 在OpenCV中，目前，它们是逐个像素地进行图像处理的三种主要方式。 为了使事情变得更有趣，将使用所有这些方法扫描每个图像，并打印出需要多长时间。

您可以在[这里](https://github.com/Itseez/opencv/tree/master/samples/cpp/tutorial_code/core/how_to_scan_images/how_to_scan_images.cpp)下载完整的源代码，或者在核心部分的cpp教程代码中的OpenCV示例目录中查找它。 其基本用法是：

```C++
how_to_scan_images imageName.jpg intValueToReduce [G]
```
最后的参数是可选的。 如果给定图像将以灰度格式加载，否则使用BGR色彩空间。 首先要计算查找表。

```C++
    int divideWith = 0; // convert our input string to number - C++ style
    stringstream s;
    s << argv[2];
    s >> divideWith;
    if (!s || !divideWith)
    {
        cout << "Invalid number entered for dividing. " << endl;
        return -1;
    }
    uchar table[256];
    for (int i = 0; i < 256; ++i)
       table[i] = (uchar)(divideWith * (i/divideWith));
```
在这里，我们首先使用C ++ stringstream类将第三个命令行参数从文本转换为整数格式。 然后我们简单的使用上面的公式来计算查找表。 在这里没有OpenCV特定的东西。

另一个问题是我们如何衡量时间？ 那么OpenCV提供了两个简单的函数来实现这个[cv::getTickCount（）](https://docs.opencv.org/3.1.0/db/de0/group__core__utils.html#gae73f58000611a1af25dd36d496bf4487)和[cv::getTickFrequency（）](https://docs.opencv.org/3.1.0/db/de0/group__core__utils.html#ga705441a9ef01f47acdc55d87fbe5090c)。 第一个从特定事件返回系统CPU的滴答数（自引导系统以来）。 第二个返回你的CPU在一秒钟内发出一个刻度的次数。 因此，以秒为单位，两次操作之间的时间间隔很容易得到：

```C++
double t = (double)getTickCount();
// do something ...
t = ((double)getTickCount() - t)/getTickFrequency();
cout << "Times passed in seconds: " << t << endl;
```
#### 图像矩阵如何存储在内存中？
正如你已经阅读我的[Mat-基本的图像容器](http://little-rocket.cn/2018/02/04/%E3%80%90OpenCvTutorials3-1%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91%E3%80%91-%20Mat-%E5%9F%BA%E6%9C%AC%E7%9A%84%E5%9B%BE%E5%83%8F%E5%AE%B9%E5%99%A8/)教程矩阵的大小取决于使用的颜色系统。 更准确地说，这取决于使用的通道数量。 在灰度图像的情况下，我们有这样的东西：

![灰度图](https://docs.opencv.org/3.1.0/tutorial_how_matrix_stored_1.png)

对于多通道图像，列包含与通道数量一样多的子列。 例如在BGR颜色系统的情况下：
![RGB颜色系统](https://docs.opencv.org/3.1.0/tutorial_how_matrix_stored_2.png)

请注意，通道的顺序是相反的：BGR而不是RGB。 因为在许多情况下，存储器足够大以便按照连续的方式存储行，所以行可以一个接一个地跟随，从而创建单个长行。 因为所有东西都在一个接一个地方，这可能有助于加快扫描过程。 我们可以使用[cv::Mat::isContinuous（）](https://docs.opencv.org/3.1.0/d3/d63/classcv_1_1Mat.html#aff83775c7fc1479de5f4a8c4e67fe361)函数来询问矩阵是否属于这种情况。 继续下一节找到一个例子。

#### 有效的方法

当谈到性能，你不能击败经典的C风格运算符[]（指针）访问。 因此，我们可以推荐的最有效的方法是：

```C++
Mat& ScanImageAndReduceC(Mat& I, const uchar* const table)
{
    // accept only char type matrices
    CV_Assert(I.depth() == CV_8U);
    int channels = I.channels();
    int nRows = I.rows;
    int nCols = I.cols * channels;
    if (I.isContinuous())
    {
        nCols *= nRows;
        nRows = 1;
    }
    int i,j;
    uchar* p;
    for( i = 0; i < nRows; ++i)
    {
        p = I.ptr<uchar>(i);
        for ( j = 0; j < nCols; ++j)
        {
            p[j] = table[p[j]];
        }
    }
    return I;
}
```
在这里，我们基本上只是获取一个指向每一行开始的指针并遍历它直到它结束。 在矩阵以连续方式存储的特殊情况下，我们只需要一次指针并一直走到最后。 我们需要注意彩色图像：我们有三个通道，所以我们需要在每一行中通过三次以上的项目(iterm)。

还有另一种方法。 Mat对象的数据数据成员将指针返回到第一行第一列。 如果这个指针为空，那么在该对象中没有有效的输入。 检查这是检查您的图像加载是否成功的最简单的方法。 如果存储是连续的，我们可以使用这个来遍历整个数据指针。 在灰度图像的情况下，这将看起来像：

```C++
uchar* p = I.data;
for( unsigned int i =0; i < ncol*nrows; ++i)
    *p++ = table[*p];
```
你会得到相同的结果。 但是，这个代码在以后很难阅读。 如果你有一些进阶的技巧，那就更难了。 此外，在实践中，我发现你会得到相同的性能结果（因为大多数现代编译器可能会自动为你做这个小优化技巧）。

#### 迭代器（安全）的方法

上一个有效的方法（efficient way）可能确定您通过适当数量的uchar字段，而且跳过行之间可能发生的差距是您的责任。 迭代器方法被认为是一个更安全的方式，因为它接管用户的这些任务。 所有你需要做的是要求图像矩阵的开始和结束，然后增加开始迭代器直到你到达最后。 要获取迭代器指向的值，请使用*运算符（在它之前添加它）。

```C++
Mat& ScanImageAndReduceIterator(Mat& I, const uchar* const table)
{
    // accept only char type matrices
    CV_Assert(I.depth() == CV_8U);
    const int channels = I.channels();
    switch(channels)
    {
    case 1:
        {
            MatIterator_<uchar> it, end;
            for( it = I.begin<uchar>(), end = I.end<uchar>(); it != end; ++it)
                *it = table[*it];
            break;
        }
    case 3:
        {
            MatIterator_<Vec3b> it, end;
            for( it = I.begin<Vec3b>(), end = I.end<Vec3b>(); it != end; ++it)
            {
                (*it)[0] = table[(*it)[0]];
                (*it)[1] = table[(*it)[1]];
                (*it)[2] = table[(*it)[2]];
            }
        }
    }
    return I;
```

在彩色图像的情况下，每列有三个uchar项目。 这可能被认为是一个short的uchar向量，它已经在OpenCV中有了[Vec3b](https://docs.opencv.org/3.1.0/dc/d84/group__core__basic.html#ga7e6060c0b8d48459964df6e1eb524c03)这个名称。 要访问第n个子列，我们使用简单的operator []访问。 记住OpenCV迭代器遍历列并自动跳到下一行是很重要的。 因此，如果使用简单的uchar迭代器，在彩色图像的情况下，您将只能访问蓝色通道值。

#### 带参考返回的实时地址计算
最后的方法不建议用于扫描。 它是为了获取或修改图像中的随机元素。 它的基本用法是指定要访问的项目的行号和列号。 在我们早期的扫描方法中，您可能已经注意到，通过我们正在查看图像的类型，这很重要。 这里没有什么不同，因为您需要手动指定在自动查找中使用的类型。 你可以在下面的源代码（+ [cv::at（）](https://docs.opencv.org/3.1.0/d5/d50/group__videostab.html#ga64715f89ad837e2b6b5649d0f833172d)函数的使用）的灰度图像的情况下观察这个：


```C++
Mat& ScanImageAndReduceRandomAccess(Mat& I, const uchar* const table)
{
    // accept only char type matrices
    CV_Assert(I.depth() == CV_8U);
    const int channels = I.channels();
    switch(channels)
    {
    case 1:
        {
            for( int i = 0; i < I.rows; ++i)
                for( int j = 0; j < I.cols; ++j )
                    I.at<uchar>(i,j) = table[I.at<uchar>(i,j)];
            break;
        }
    case 3:
        {
         Mat_<Vec3b> _I = I;
         for( int i = 0; i < I.rows; ++i)
            for( int j = 0; j < I.cols; ++j )
               {
                   _I(i,j)[0] = table[_I(i,j)[0]];
                   _I(i,j)[1] = table[_I(i,j)[1]];
                   _I(i,j)[2] = table[_I(i,j)[2]];
            }
         I = _I;
         break;
        }
    }
    return I;
}
```
这些功能将输入您的输入类型和坐标，并实时计算查询项目的地址。 然后返回一个参考。 当您get该值时，这可能是一个常量，而当您set该值时，该值可能是不恒定的。 作为仅在debug模式下的安全步骤*将执行一次检查，确认您的输入坐标是有效的并且确实存在。 如果不是这种情况，你会在标准错误输出流中得到一个很好的输出消息。 与release模式中的高效方式相比，使用这种方法的唯一区别是，对于图像的每个元素，您将获得一个新的行指针，用于我们使用C运算符[]获取列元素。

如果您需要使用此方法对图像进行多次查找，那么输入每个访问的类型和at关键字可能会很麻烦并且非常耗时。 为了解决这个问题，OpenCV有一个[cv::Mat_](https://docs.opencv.org/3.1.0/df/dfc/classcv_1_1Mat__.html)数据类型。 与Mat相同，额外的需求是在定义时需要通过查看数据矩阵来指定数据类型，但是作为回报，您可以使用operator（）来快速访问项目。 为了使事情更好，这很容易转换到通常的[cv :: Mat](https://docs.opencv.org/3.1.0/d3/d63/classcv_1_1Mat.html)数据类型。 上面的函数的彩色图像的情况下，您可以看到一个示例用法。 不过，需要注意的是，使用[cv::at（）](https://docs.opencv.org/3.1.0/d5/d50/group__videostab.html#ga64715f89ad837e2b6b5649d0f833172d)函数可以完成相同的操作（具有相同的运行时速度）。 这只是懒惰的程序员技巧写的。


#### 核心函数
这是一个在图像中实现查找表修改的额外方法。 因为在图像处理中，您想要将所有给定的图像值替换为其他值，这是相当普遍的。OpenCV具有一个功能，无需编写图像扫描即可进行修改。 我们使用核心模块的[cv::LUT（）](https://docs.opencv.org/3.1.0/d2/de8/group__core__array.html#gab55b8d062b7f5587720ede032d34156f)函数。 首先我们建立一个Mat查找表的类型：

```C++
    Mat lookUpTable(1, 256, CV_8U);
    uchar* p = lookUpTable.ptr();
    for( int i = 0; i < 256; ++i)
        p[i] = table[i];
```

最后调用函数（我是我们的输入图像，J是输出图像）：

```C++
 LUT(I, lookUpTable, J);
```

#### 性能差异
为了得到最好的结果，编译程序并以你自己的速度运行。 为了展示更好的差异，我使用了一个相当大的（2560 X 1600）图像。 这里介绍的性能是彩色图像。 为了获得更准确的值，我已经从函数调用中获得了数百次的平均值。

| 方法     | 时间   |
| :------- |  :---: |
| 高效的方式 |  79.4717毫秒     |
| 迭代器    | 83.7201毫秒   |
| 实时性计算     |  93.7878毫秒   |
| LUT函数| 32.5759毫秒|

总结一下。 如果可能的话，使用OpenCV已经完成的功能（而不是重新创建这些功能）。 最快的方法就是LUT功能。 这是因为OpenCV库是通过Intel Threaded Building Blocks启用的多线程。 但是，如果您需要编写一个简单的图像扫描更喜欢指针的方法。 迭代器是一个更安全的，但相当慢。 在Debug模式下，使用实时参考访问方法进行全图像扫描是成本最高的。 在release模式下，它可能会打败迭代器的方法，但它肯定会牺牲迭代器的安全特性。

最后，您可以在[YouTube频道](https://www.youtube.com/watch?v=fB3AN5fjgwc)上发布的视频中观看该节目的示例。

<iframe width="560" height="349" src="https://www.youtube.com/embed/fB3AN5fjgwc" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>