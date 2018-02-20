---
title: 【OpenCvTutorials3.1中文翻译】- 在矩阵上进行掩码操作
date: 2018-02-05 13:10:49
tags: [OpenCvTutorials3.1中文翻译]
categories: [原创]
thumbnail:  https://docs.opencv.org/3.1.0/resultMatMaskFilter2D.png
toc: true
---
### [Mask operations on matrices](https://docs.opencv.org/3.1.0/d7/d37/tutorial_mat_mask_operations.html)

矩阵上的[掩码操作](https://www.baidu.com/s?wd=%E7%9F%A9%E9%98%B5%E7%9A%84%E6%8E%A9%E7%A0%81%E6%93%8D%E4%BD%9C&rsv_spt=1&rsv_iqid=0xa869e59200027f86&issp=1&f=8&rsv_bp=0&rsv_idx=2&ie=utf-8&rqlang=&tn=baiduhome_pg&ch=&rsv_enter=0&inputT=1058)非常简单。 我们的想法是，我们根据掩码矩阵（也称为内核）重新计算图像中的每个像素值。 该掩码保存的值将调整相邻像素（和当前像素）对新像素值的影响程度。 从数学的角度来看，我们用我们指定的数值进行加权平均。

<!-- more -->

<script type="text/javascript" async
  src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.2/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

#### 我们的测试案例

让我们考虑一个图像对比度增强方法的问题。 基本上我们要使用下列公式申请图像的每个像素：

<script type="text/javascript" async
  src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.2/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

![公式](http://otneosm59.bkt.clouddn.com/18_2_5.png)

第一种表示法是使用公式，而第二种表示法是使用掩码的第一种表示法。 通过将掩码矩阵的中心（在由零 - 零指数标记的大写字母中）放在要计算的像素上并使用与重叠的矩阵值相乘的像素值相加来使用掩码。 这是一样的事情，但是在大矩阵的情况下，后面的符号更容易查找。

现在让我们看看如何使用基本像素访问方法或使用[cv::filter2D](https://docs.opencv.org/3.1.0/d4/d86/group__imgproc__filter.html#ga27c049795ce870216ddfb366086b5a04)函数来实现这一点。

#### 基本方法

函数的实现：

```C++
void Sharpen(const Mat& myImage, Mat& Result)
{
    CV_Assert(myImage.depth() == CV_8U);  // accept only uchar images
    Result.create(myImage.size(), myImage.type());
    const int nChannels = myImage.channels();
    for(int j = 1; j < myImage.rows - 1; ++j)
    {
        const uchar* previous = myImage.ptr<uchar>(j - 1);
        const uchar* current  = myImage.ptr<uchar>(j    );
        const uchar* next     = myImage.ptr<uchar>(j + 1);
        uchar* output = Result.ptr<uchar>(j);
        for(int i = nChannels; i < nChannels * (myImage.cols - 1); ++i)
        {
            *output++ = saturate_cast<uchar>(5 * current[i]
                         -current[i - nChannels] - current[i + nChannels] - previous[i] - next[i]);
        }
    }
    Result.row(0).setTo(Scalar(0));
    Result.row(Result.rows - 1).setTo(Scalar(0));
    Result.col(0).setTo(Scalar(0));
    Result.col(Result.cols - 1).setTo(Scalar(0));
}
```
首先我们确保输入的图像数据是无符号的字符格式。 为此，我们使用[cv::CV_Assert](https://docs.opencv.org/3.1.0/db/de0/group__core__utils.html#gaf62bcd90f70e275191ab95136d85906b)函数，当它里面的表达式为false时会抛出一个错误。

```C++
CV_Assert(myImage.depth() == CV_8U);  // accept only uchar images
```
我们创建一个与我们的输入具有相同大小和相同类型的输出图像。 正如你可以在[图像存储部分](http://little-rocket.cn/2018/02/04/%E3%80%90OpenCvTutorials3-1%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91%E3%80%91-%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8OpenCV%E6%89%AB%E6%8F%8F%E5%9B%BE%E5%83%8F%EF%BC%8C%E6%9F%A5%E6%89%BE%E8%A1%A8%E5%92%8C%E6%97%B6%E9%97%B4%E6%B5%8B%E9%87%8F/)看到的，取决于通道的数量，我们可能有一个或多个子列。 我们将通过指针遍历它们，所以元素的总数取决于这个数字。

```C++
Result.create(myImage.size(), myImage.type());
const int nChannels = myImage.channels();
```

我们将使用普通的C []运算符来访问像素。 因为我们需要同时访问多行，我们将获取每个行的指针（前一行，当前行和下一行）。 我们需要另一个指向我们要保存计算的地方。 然后只需使用[]运算符访问正确的项目。 为了提前移动输出指针，我们只需在每个操作之后增加一个字节即可：

```C++
for(int j = 1; j < myImage.rows - 1; ++j)
{
    const uchar* previous = myImage.ptr<uchar>(j - 1);
    const uchar* current  = myImage.ptr<uchar>(j    );
    const uchar* next     = myImage.ptr<uchar>(j + 1);

    uchar* output = Result.ptr<uchar>(j);

    for(int i = nChannels; i < nChannels * (myImage.cols - 1); ++i)
    {
        *output++ = saturate_cast<uchar>(5 * current[i]
                     -current[i - nChannels] - current[i + nChannels] - previous[i] - next[i]);
    }
}
```
在图像的边界上面的符号导致不存在的像素位置例如(-1,-1)`(like minus one - minus one).`。 在这点我们的公式是不确定的。 一个简单的解决方案是不在这些点上应用内核，例如，将边界上的像素设置为零：
> 译者注：
> "在图像的边界上面的符号导致不存在的像素位置例如(-1,-1)`(like minus one - minus one)."`在原文是“On the borders of the image the upper notation results inexistent pixel locations (like minus one - minus one). ”不好翻译，需要结合下面的代码理解一下，大概就是有些地方遍历不到，故直接进行赋值。


```C++
Result.row(0).setTo(Scalar(0));               // The top row
Result.row(Result.rows - 1).setTo(Scalar(0)); // The bottom row
Result.col(0).setTo(Scalar(0));               // The left column
Result.col(Result.cols - 1).setTo(Scalar(0)); // The right column
```
#### filter2D函数

应用这样的过滤器在图像处理中非常普遍，以至于在OpenCV中存在一个将处理掩码（在某些地方也称为内核）的函数。 为此，你首先需要定义一个Mat对象来保存这个掩码：

```C++
Mat kern = (Mat_<char>(3,3) <<  0, -1,  0,
                               -1,  5, -1,
                                0, -1,  0);
```

然后调用[cv::filter2D](https://docs.opencv.org/3.1.0/d4/d86/group__imgproc__filter.html#ga27c049795ce870216ddfb366086b5a04)函数指定输入: 输出的图像和使用的内核

```C++
filter2D(I, K, I.depth(), kern);
```
该函数甚至有第五个可选参数来指定内核的中心，第六个用于确定在未定义操作（边界）的区域中要做什么。 使用这个函数的优点是它更短，更简洁，并且由于实现了一些优化技术，所以通常比手工编码的方法更快。 例如在我的测试中，第二个只花了13毫秒，第一个花费了大约31毫秒。 有一些差异。

For example:

![例子](https://docs.opencv.org/3.1.0/resultMatMaskFilter2D.png)

你可以从[这里](https://github.com/Itseez/opencv/tree/master/samples/cpp/tutorial_code/core/mat_mask_operations/mat_mask_operations.cpp)下载这个源代码，或者查看OpenCV源代码库的示例目录

`samples/cpp/tutorial_code/core/mat_mask_operations/mat_mask_operations.cpp.`

查看在我们的[YouTube频道](https://youtube.com/watch?v=7PF1tAU9se4)上运行该程序的实例。
<iframe width="560" height="349" src="https://www.youtube.com/embed/7PF1tAU9se4" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>