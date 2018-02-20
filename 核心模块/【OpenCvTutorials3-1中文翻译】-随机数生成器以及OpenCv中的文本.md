---
title: 【OpenCvTutorials3.1中文翻译】- 随机数生成器以及OpenCv中的文本
date: 2018-02-10 19:49:08
tags: [OpenCvTutorials3.1中文翻译]
categories: [原创]
thumbnail: http://image.little-rocket.cn/Drawing_2_Tutorial_Result_big.jpg
toc: true
---
<!-- Go to www.addthis.com/dashboard to customize your tools -->
<script type="text/javascript" src="//s7.addthis.com/js/300/addthis_widget.js#pubid=ra-59cb1e1aae525c27"></script>
### [Random generator and text with OpenCV](https://docs.opencv.org/3.1.0/df/d61/tutorial_random_generator_and_text.html)

#### Goal

在本教程中，您将学习：
* 使用随机数生成器类[（cv::RNG）](https://docs.opencv.org/3.1.0/d1/dd6/classcv_1_1RNG.html)以及如何从统一分布中获得一个随机数。
* 使用函数[cv::putText](https://docs.opencv.org/3.1.0/d6/d6e/group__imgproc__draw.html#ga5126f47f883d730f633d74f07456c576)在OpenCV窗口上显示文本

<!-- more -->
#### Code
* 在前面的教程（[Basic Drawing](http://little-rocket.cn/2018/02/06/%E3%80%90OpenCvTutorials3-1%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91%E3%80%91-%E5%9F%BA%E6%9C%AC%E7%BB%98%E5%9B%BE/)）中，我们绘制了不同的几何图形，给出了输入参数，如坐标（以cv :: Point的形式），颜色，厚度等等。您可能已经注意到，我们给出的是这些参数的具体值。

* 在本教程中我们打算使用随机值作为绘图函数的参数。此外，我们打算用大量的几何图形填充我们的图像。由于我们将以随机方式初始化它们，所以这个过程将会通过循环来自动的。
* 此代码位于您的OpenCV的sample文件夹中。 否则，你可以从[这里](http://code.opencv.org/projects/opencv/repository/revisions/master/raw/samples/cpp/tutorial_code/core/Matrix/Drawing_2.cpp)得到它。

#### Explanation
1、我们先看看main函数。 我们观察到，我们所做的第一件事是创建一个随机数生成器对象（RNG）：
```C++
RNG rng( 0xFFFFFFFF );
```
RNG实现一个随机数字发生器。 在这个例子中，rng是一个用值0xFFFFFFFF初始化的RNG元素

2、然后我们创建一个初始化为零的矩阵（这意味着它将显示为黑色），并且指定其高度，宽度和类型：
```C++
Mat image = Mat::zeros( window_height, window_width, CV_8UC3 );
imshow( window_name, image );
```
3、然后我们开始疯狂的作图， 看过代码之后， 可以看到它主要分为8个部分， 并且被定义为以下函数：
```C++
c = Drawing_Random_Lines(image, window_name, rng);
if( c != 0 ) return 0;
c = Drawing_Random_Rectangles(image, window_name, rng);
if( c != 0 ) return 0;
c = Drawing_Random_Ellipses( image, window_name, rng );
if( c != 0 ) return 0;
c = Drawing_Random_Polylines( image, window_name, rng );
if( c != 0 ) return 0;
c = Drawing_Random_Filled_Polygons( image, window_name, rng );
if( c != 0 ) return 0;
c = Drawing_Random_Circles( image, window_name, rng );
if( c != 0 ) return 0;
c = Displaying_Random_Text( image, window_name, rng );
if( c != 0 ) return 0;
c = Displaying_Big_End( image, window_name, rng );
```
所有这些功能都遵循相同的模式，所以我们只分析其中的一个，因为所有的说明几乎都是一样的。
4、研究一下函数 ** Drawing_Random_Lines **：
```C++
int Drawing_Random_Lines( Mat image, char* window_name, RNG rng )
{
  int lineType = 8;
  Point pt1, pt2;
  for( int i = 0; i < NUMBER; i++ )
  {
   pt1.x = rng.uniform( x_1, x_2 );
   pt1.y = rng.uniform( y_1, y_2 );
   pt2.x = rng.uniform( x_1, x_2 );
   pt2.y = rng.uniform( y_1, y_2 );
   line( image, pt1, pt2, randomColor(rng), rng.uniform(1, 10), 8 );
   imshow( window_name, image );
   if( waitKey( DELAY ) >= 0 )
   { return -1; }
  }
  return 0;
}
```
我们可以知道：
* for循环将重复 **NUMBER** 次。 由于函数[cv::line](https://docs.opencv.org/3.1.0/d6/d6e/group__imgproc__draw.html#ga7078a9fae8c7e7d13d24dac2520ae4a2)在这个循环中，这意味着将会生成 **NUMBER**条`line`。

* 线的端点是pt1和pt2提供的。 对于pt1我们可以看到：
```C++
pt1.x = rng.uniform( x_1, x_2 );
pt1.y = rng.uniform( y_1, y_2 );
```
- *我们知道rng是一个随机数字生成器对象。 在上面的代码中，我们调用 **rng.uniform（a，b）**。 这会在值 **a** 和 **b** 之间产生一个随机均匀分布（包括 **a**，不包括 **b**）。

- 从上面的解释中，我们推断极值pt1和pt2将是随机值，所以线的位置将是相当不可预测的，给出了一个很好的视觉效果（查看下面的Result部分）。
- 另一方面，我们注意到在[cv::line](https://docs.opencv.org/3.1.0/d6/d6e/group__imgproc__draw.html#ga7078a9fae8c7e7d13d24dac2520ae4a2)参数中，对于颜色我们输入：
```C++
randomColor(rng)
```
我们来看看函数的实现：
```C++
static Scalar randomColor( RNG& rng )
  {
    int icolor = (unsigned) rng;
    return Scalar( icolor&255, (icolor>>8)&255, (icolor>>16)&255 );
  }
```
我们可以看到，返回值是一个具有3个随机初始化值的标量，用作线颜色的R，G和B参数。 因此，线条的颜色也是随机的！

5、上面的解释适用于生成圆，椭圆，多边形等其他函数。中心和顶点等参数也是随机生成的。

6、在完成之前，我们还应该看看函数 **Display_Random_Text** 和 **Displaying_Big_End**，因为它们都有一些有趣的特性：

7、**Display_Random_Text** 
```C++
int Displaying_Random_Text( Mat image, char* window_name, RNG rng )
{
  int lineType = 8;
  for ( int i = 1; i < NUMBER; i++ )
  {
    Point org;
    org.x = rng.uniform(x_1, x_2);
    org.y = rng.uniform(y_1, y_2);
    putText( image, "Testing text rendering", org, rng.uniform(0,8),
             rng.uniform(0,100)*0.05+0.1, randomColor(rng), rng.uniform(1, 10), lineType);
    imshow( window_name, image );
    if( waitKey(DELAY) >= 0 )
      { return -1; }
  }
  return 0;
}
```
一切看起来很熟悉，但表达的：
```C++
putText( image, "Testing text rendering", org, rng.uniform(0,8),
         rng.uniform(0,100)*0.05+0.1, randomColor(rng), rng.uniform(1, 10), lineType);
```

那么，在我们的例子中函数[cv::putText](https://docs.opencv.org/3.1.0/d6/d6e/group__imgproc__draw.html#ga5126f47f883d730f633d74f07456c576)做了什么？ 
    * 在图像中绘制文本  “Testing text rendering” 
    * 文本的左下角将位于 `Point org`.
    * 字体类型(font type)是一个随机的整数值，范围：[0,8)
    * 字体的尺度用表达式 `rng.uniform（0,100）x0.05 + 0.1`（表示它的范围是：[0.1,5.1)）
    * 文本颜色是随机的（由 `randomColor（rng）`表示）
    * 文本厚度在1到10之间，由rng.uniform（1,10）指定。

因此，我们将在随机位置得到（与其他绘图功能类似的）**NUMBER**个文本。

8、 **Displaying_Big_End**
```C++
int Displaying_Big_End( Mat image, char* window_name, RNG rng )
{
  Size textsize = getTextSize("OpenCV forever!", FONT_HERSHEY_COMPLEX, 3, 5, 0);
  Point org((window_width - textsize.width)/2, (window_height - textsize.height)/2);
  int lineType = 8;
  Mat image2;
  for( int i = 0; i < 255; i += 2 )
  {
    image2 = image - Scalar::all(i);
    putText( image2, "OpenCV forever!", org, FONT_HERSHEY_COMPLEX, 3,
           Scalar(i, i, 255), 5, lineType );
    imshow( window_name, image2 );
    if( waitKey(DELAY) >= 0 )
      { return -1; }
  }
  return 0;
}
```
除了函数getTextSize（获取参数文本的大小），我们可以观察到的新操作是在for循环内：
```C++
image2 = image - Scalar::all(i)
```
因此， **image2** 是 **image** 和 **Scalar::all（i）**的减法运算， 实际上，这里发生的事情是：**image2**的每一个像素将是 **image**的每一个像素减去 **i**的结果（请记住，对于每个像素，我们考虑三个值：R，G和B，因此每个像素都将受到影响）。

另外请记住，减法运算总是在内部执行一个饱和运算，这意味着得到的结果总是在允许的范围内（对我们的例子来说，没有负数，都在0到255之间）。

#### Result
正如你在代码部分看到的，程序将顺序执行不同的绘图功能，这将产生：

1、首先随机设置的 **NUMBER** lines 将出现在屏幕上，如下图所示：
![lines](http://image.little-rocket.cn/Drawing_2_Tutorial_Result_0.jpg)

2、然后，一组新的图形，这些时间矩形将随之而来。
3、现在会出现一些椭圆，每个椭圆都是随机的位置，大小，厚度和弧长：
![Result_2](http://image.little-rocket.cn/Drawing_2_Tutorial_Result_2.jpg)

4、现在，03段的折线将出现在屏幕上，也是随机配置。
![Result_3](http://image.little-rocket.cn/Drawing_2_Tutorial_Result_3.jpg)

5、填充的多边形（在这个例子中是三角形）随之而来。
6、最后要显示的一个几何图形：圆圈！
![Result_4](http://image.little-rocket.cn/Drawing_2_Tutorial_Result_5.jpg)

7、紧接着最后，文本*“Testing Text Rendering”*将出现在各种字体，大小，颜色和位置。

8、而大的结局（顺便也表达了一个大真理）：
![Result_big](http://image.little-rocket.cn/Drawing_2_Tutorial_Result_big.jpg)

