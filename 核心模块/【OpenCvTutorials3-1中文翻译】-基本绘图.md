---
title: 【OpenCvTutorials3.1中文翻译】- 基本绘图
date: 2018-02-06 17:15:15
tags: [OpenCvTutorials3.1中文翻译]
categories: [原创]
thumbnail:  https://docs.opencv.org/3.1.0/Drawing_1_Tutorial_Result_0.png
toc: true
---

### [Basic Drawing](https://docs.opencv.org/3.1.0/d3/d96/tutorial_basic_geometric_drawing.html)
<!-- Go to www.addthis.com/dashboard to customize your tools -->
<script type="text/javascript" src="//s7.addthis.com/js/300/addthis_widget.js#pubid=ra-59cb1e1aae525c27"></script>
#### Golas:

在本教程中，您将学习如何：

* 使用[cv::Point](https://docs.opencv.org/3.1.0/dc/d84/group__core__basic.html#ga1e83eafb2d26b3c93f09e8338bcab192)在图像中定义2D点。
* 使用[cv::Scalar](https://docs.opencv.org/3.1.0/dc/d84/group__core__basic.html#ga599fe92e910c027be274233eccad7beb)和它为什么是有用的
* 使用OpenCV函数[cv::line](https://docs.opencv.org/3.1.0/d6/d6e/group__imgproc__draw.html#ga7078a9fae8c7e7d13d24dac2520ae4a2)绘制一条线
* 使用OpenCV函数[cv::ellipse](https://docs.opencv.org/3.1.0/d6/d6e/group__imgproc__draw.html#ga28b2267d35786f5f890ca167236cbc69)绘制一个椭圆
* 使用OpenCV函数[cv::rectangle](https://docs.opencv.org/3.1.0/d6/d6e/group__imgproc__draw.html#ga07d2f74cadcf8e305e810ce8eed13bc9)绘制一个矩形
* 使用OpenCV函数[cv::circle](https://docs.opencv.org/3.1.0/d6/d6e/group__imgproc__draw.html#gaf10604b069374903dbd0f0488cb43670)绘制一个圆
* 使用OpenCV函数[cv::fillPoly](https://docs.opencv.org/3.1.0/d6/d6e/group__imgproc__draw.html#gaf30888828337aa4c6b56782b5dfbd4b7)绘制一个填充的多边形
<!-- more -->
#### OpenCV Theory
对于本教程，我们将大量使用两个结构:[cv::Point](https://docs.opencv.org/3.1.0/dc/d84/group__core__basic.html#ga1e83eafb2d26b3c93f09e8338bcab192)和[cv::Scalar](https://docs.opencv.org/3.1.0/dc/d84/group__core__basic.html#ga599fe92e910c027be274233eccad7beb)

##### Point

它表示一个2D点，由它的图像坐标`x、y`指定:
```C++
Point pt;
pt.x = 10;
pt.y = 8;
```
或者
```C++
Point pt =  Point(10, 8);
```

##### Scalar

* 代表一个4元素的向量。 标量类型广泛用于OpenCV传递像素值。
* 在本教程中，我们将广泛使用它来表示BGR颜色值（3个参数）。 如果不打算使用最后一个参数，则没有必要定义它。
* 让我们看一个例子，如果我们需要一个颜色参数，我们设定：
```C++
Scalar( a, b, c )
```
我们将定义BGR颜色，例如：蓝色= a，绿色= b和红色= c

#### Code
此代码位于您的OpenCV示例文件夹中。 此外，你可以从[这里](https://github.com/Itseez/opencv/tree/master/samples/cpp/tutorial_code/core/Matrix/Drawing_1.cpp)找到它。

#### Explanation

1、既然我们打算画两个例子（一个atom和一个rook），nam我们必须创建02个图像和两个窗口来显示它们。

```C++
char atom_window[] = "Drawing 1: Atom";
char rook_window[] = "Drawing 2: Rook";
Mat atom_image = Mat::zeros( w, w, CV_8UC3 );
Mat rook_image = Mat::zeros( w, w, CV_8UC3 );
```

2、我们创建了绘制不同几何形状的函数。 例如，要绘制Atom我们使用MyEllipse和MyFilledCircle：

```C++
MyEllipse( atom_image, 90 );
MyEllipse( atom_image, 0 );
MyEllipse( atom_image, 45 );
MyEllipse( atom_image, -45 );
MyFilledCircle( atom_image, Point( w/2.0, w/2.0) );
```
3、为了画出我们的rook的我们使用 `MyLine`，`rectangle `和`MyPolygon`：
```C++
MyPolygon( rook_image );
rectangle( rook_image,
       Point( 0, 7*w/8.0 ),
       Point( w, w),
       Scalar( 0, 255, 255 ),
       -1,
       8 );
MyLine( rook_image, Point( 0, 15*w/16 ), Point( w, 15*w/16 ) );
MyLine( rook_image, Point( w/4, 7*w/8 ), Point( w/4, w ) );
MyLine( rook_image, Point( w/2, 7*w/8 ), Point( w/2, w ) );
MyLine( rook_image, Point( 3*w/4, 7*w/8 ), Point( 3*w/4, w ) );
```
4、让我们来看看每个函数里面的内容：

##### MyLine
```C++
void MyLine( Mat img, Point start, Point end )
{
    int thickness = 2;
    int lineType = 8;
    line( img, start, end,
          Scalar( 0, 0, 0 ),
          thickness,
          lineType );
}
```
正如我们所看到的，MyLine只是调用函数cv :: line，它执行以下操作：
* 从`start`点到`end`点画一条线
* 这个直线显示在图像img中
* 线的颜色由`Scalar( 0, 0, 0)`定义，这是与黑色对应的RGB值
* 线的厚度被设定为`thickness`（在这种情况下为2）
* 这条线是一个8连接的（lineType = 8）
---


##### MyEllipse
```C++
void MyEllipse( Mat img, double angle )
{
    int thickness = 2;
    int lineType = 8;
    ellipse( img,
       Point( w/2.0, w/2.0 ),
       Size( w/4.0, w/16.0 ),
       angle,
       0,
       360,
       Scalar( 255, 0, 0 ),
       thickness,
       lineType );
}
```
从上面的代码，我们可以观察到函数[cv::ellipse](https://docs.opencv.org/3.1.0/d6/d6e/group__imgproc__draw.html#ga28b2267d35786f5f890ca167236cbc69)绘制一个椭圆，使得：

椭圆显示在图像**img**中
* 椭圆中心位于点 **（w / 2.0，w / 2.0）** 中，并被包围在一个大小为 **（w / 4.0，w / 16.0）** 的框中
* 椭圆旋转**angle**角度
* 椭圆在**0**和**360**度之间延伸一个弧
* 图形的颜色将是**Scalar（255，0，0）**，这意味着RGB值为蓝色。
* 椭圆的**thickness**是2。

---
##### MyFilledCircle
```C++
void MyFilledCircle( Mat img, Point center )
{
    int thickness = -1;
    int lineType = 8;
    circle( img,
        center,
        w/32.0,
        Scalar( 0, 0, 255 ),
        thickness,
        lineType );
}
```
类似于椭圆函数，我们可以观察到`circle`作为参数接收：
将显示圆圈的图像**（img）**
* 圆的中心表示为点**center**
* 圆的半径：**w / 32.0**
* 圆的颜色：**Scalar（0，0，255）**，意思是BGR中的红色
* 由于厚度 **(thickness)** = -1，圆圈将被填充。
---

##### MyPolygon
```C++
void MyPolygon( Mat img )
{
    int lineType = 8;
    /* Create some points */
    Point rook_points[1][20];
    rook_points[0][0] = Point( w/4.0, 7*w/8.0 );
    rook_points[0][1] = Point( 3*w/4.0, 7*w/8.0 );
    rook_points[0][2] = Point( 3*w/4.0, 13*w/16.0 );
    rook_points[0][3] = Point( 11*w/16.0, 13*w/16.0 );
    rook_points[0][4] = Point( 19*w/32.0, 3*w/8.0 );
    rook_points[0][5] = Point( 3*w/4.0, 3*w/8.0 );
    rook_points[0][6] = Point( 3*w/4.0, w/8.0 );
    rook_points[0][7] = Point( 26*w/40.0, w/8.0 );
    rook_points[0][8] = Point( 26*w/40.0, w/4.0 );
    rook_points[0][9] = Point( 22*w/40.0, w/4.0 );
    rook_points[0][10] = Point( 22*w/40.0, w/8.0 );
    rook_points[0][11] = Point( 18*w/40.0, w/8.0 );
    rook_points[0][12] = Point( 18*w/40.0, w/4.0 );
    rook_points[0][13] = Point( 14*w/40.0, w/4.0 );
    rook_points[0][14] = Point( 14*w/40.0, w/8.0 );
    rook_points[0][15] = Point( w/4.0, w/8.0 );
    rook_points[0][16] = Point( w/4.0, 3*w/8.0 );
    rook_points[0][17] = Point( 13*w/32.0, 3*w/8.0 );
    rook_points[0][18] = Point( 5*w/16.0, 13*w/16.0 );
    rook_points[0][19] = Point( w/4.0, 13*w/16.0) ;
    const Point* ppt[1] = { rook_points[0] };
    int npt[] = { 20 };
    fillPoly( img,
              ppt,
              npt,
                  1,
              Scalar( 255, 255, 255 ),
              lineType );
}
```
要绘制一个填充的多边形，我们使用函数[cv::fillPoly](https://docs.opencv.org/3.1.0/d6/d6e/group__imgproc__draw.html#gaf30888828337aa4c6b56782b5dfbd4b7)。 我们注意到：

多边形将在 **img** 上绘制
* 多边形的顶点是 **ppt**中的点的集合
* 要绘制的顶点总数是 **npt**
* 要绘制的多边形数量只有 **1**个
* 多边形的颜色由 **Scalar（255,255,255）**定义，它是白色的BGR值

---
##### rectangle

```C++
rectangle( rook_image,
           Point( 0, 7*w/8.0 ),
           Point( w, w),
           Scalar( 0, 255, 255 ),
           -1, 8 );
```
最后我们有[cv::rectangle](https://docs.opencv.org/3.1.0/d6/d6e/group__imgproc__draw.html#ga07d2f74cadcf8e305e810ce8eed13bc9)函数（我们没有为这个人创建一个特殊的函数）。 我们注意到：
* 矩形将在 **rook_image**上绘制
* **Point（0，7 * w / 8.0）** 和 **Point（w，w）**定义矩形的两个相对顶点
* 矩形的颜色由Scalar（0,255,255）给出，它是黄色的BGR值
* 由于厚度值由 **- 1** 给出，矩形将被填充


#### Result
编译和运行你的程序应该给你这样的结果：
 ![Result](https://docs.opencv.org/3.1.0/Drawing_1_Tutorial_Result_0.png)