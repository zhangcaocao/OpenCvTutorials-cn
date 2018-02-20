---
title: 【OpenCvTutorials3.1中文翻译】- 使用XML和YAML文件的文件输入和输出
toc: true
thumbnail: 'http://image.little-rocket.cn/18_02_17_01.PNG'
date: 2018-02-16 19:01:49
tags: [OpenCvTutorials3.1中文翻译]
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

### [File Input and Output using XML and YAML files](https://docs.opencv.org/3.1.0/dd/d74/tutorial_file_input_output_with_xml_yml.html)

#### Goal:
你会找到以下问题的答案：

* 如何使用YAML或XML文件将文本条目打印并读取到文件和OpenCV？
* 如何为OpenCV数据结构做同样的事情？
* 如何为你的数据结构做到这一点？
* OpenCV的数据结构，例如[CV::FileStorage](https://docs.opencv.org/3.1.0/da/d56/classcv_1_1FileStorage.html)，[CV::filenode](https://docs.opencv.org/3.1.0/de/dd9/classcv_1_1FileNode.html) 的或[CV::FileNodeIterator](https://docs.opencv.org/3.1.0/d7/d4e/classcv_1_1FileNodeIterator.html)的使用。
<!-- more -->

#### Source code:
你可以从[这里](https://github.com/Itseez/opencv/tree/master/samples/cpp/tutorial_code/core/file_input_output/file_input_output.cpp)下载， 也可以在`samples/cpp/tutorial_code/core/file_input_output/file_input_output.cpp`找到：
以下是如何实现目标列表中列举的所有东西的示例代码。
```C++
#include <opencv2/core/core.hpp>
#include <iostream>
#include <string>
using namespace cv;
using namespace std;
static void help(char** av)
{
    cout << endl
        << av[0] << " shows the usage of the OpenCV serialization functionality."         << endl
        << "usage: "                                                                      << endl
        <<  av[0] << " outputfile.yml.gz"                                                 << endl
        << "The output file may be either XML (xml) or YAML (yml/yaml). You can even compress it by "
        << "specifying this in its extension like xml.gz yaml.gz etc... "                  << endl
        << "With FileStorage you can serialize objects in OpenCV by using the << and >> operators" << endl
        << "For example: - create a class and have it serialized"                         << endl
        << "             - use it to read and write matrices."                            << endl;
}
class MyData
{
public:
    MyData() : A(0), X(0), id()
    {}
    explicit MyData(int) : A(97), X(CV_PI), id("mydata1234") // explicit to avoid implicit conversion
    {}
    void write(FileStorage& fs) const                        //Write serialization for this class
    {
        fs << "{" << "A" << A << "X" << X << "id" << id << "}";
    }
    void read(const FileNode& node)                          //Read serialization for this class
    {
        A = (int)node["A"];
        X = (double)node["X"];
        id = (string)node["id"];
    }
public:   // Data Members
    int A;
    double X;
    string id;
};
//These write and read functions must be defined for the serialization in FileStorage to work
static void write(FileStorage& fs, const std::string&, const MyData& x)
{
    x.write(fs);
}
static void read(const FileNode& node, MyData& x, const MyData& default_value = MyData()){
    if(node.empty())
        x = default_value;
    else
        x.read(node);
}
// This function will print our custom class to the console
static ostream& operator<<(ostream& out, const MyData& m)
{
    out << "{ id = " << m.id << ", ";
    out << "X = " << m.X << ", ";
    out << "A = " << m.A << "}";
    return out;
}
int main(int ac, char** av)
{
    if (ac != 2)
    {
        help(av);
        return 1;
    }
    string filename = av[1];
    { //write
        Mat R = Mat_<uchar>::eye(3, 3),
            T = Mat_<double>::zeros(3, 1);
        MyData m(1);
        FileStorage fs(filename, FileStorage::WRITE);
        fs << "iterationNr" << 100;
        fs << "strings" << "[";                              // text - string sequence
        fs << "image1.jpg" << "Awesomeness" << "../data/baboon.jpg";
        fs << "]";                                           // close sequence
        fs << "Mapping";                              // text - mapping
        fs << "{" << "One" << 1;
        fs <<        "Two" << 2 << "}";
        fs << "R" << R;                                      // cv::Mat
        fs << "T" << T;
        fs << "MyData" << m;                                // your own data structures
        fs.release();                                       // explicit close
        cout << "Write Done." << endl;
    }
    {//read
        cout << endl << "Reading: " << endl;
        FileStorage fs;
        fs.open(filename, FileStorage::READ);
        int itNr;
        //fs["iterationNr"] >> itNr;
        itNr = (int) fs["iterationNr"];
        cout << itNr;
        if (!fs.isOpened())
        {
            cerr << "Failed to open " << filename << endl;
            help(av);
            return 1;
        }
        FileNode n = fs["strings"];                         // Read string sequence - Get node
        if (n.type() != FileNode::SEQ)
        {
            cerr << "strings is not a sequence! FAIL" << endl;
            return 1;
        }
        FileNodeIterator it = n.begin(), it_end = n.end(); // Go through the node
        for (; it != it_end; ++it)
            cout << (string)*it << endl;
        n = fs["Mapping"];                                // Read mappings from a sequence
        cout << "Two  " << (int)(n["Two"]) << "; ";
        cout << "One  " << (int)(n["One"]) << endl << endl;
        MyData m;
        Mat R, T;
        fs["R"] >> R;                                      // Read cv::Mat
        fs["T"] >> T;
        fs["MyData"] >> m;                                 // Read your own structure_
        cout << endl
            << "R = " << R << endl;
        cout << "T = " << T << endl << endl;
        cout << "MyData = " << endl << m << endl << endl;
        //Show default behavior for non existing nodes
        cout << "Attempt to read NonExisting (should initialize the data structure with its default).";
        fs["NonExisting"] >> m;
        cout << endl << "NonExisting = " << endl << m << endl;
    }
    cout << endl
        << "Tip: Open up " << filename << " with a text editor to see the serialized data." << endl;
    return 0;
}
```

#### Explanation:

这里我们只讨论XML和YAML文件输入。您的输出（及其各自的输入）文件可能只有其中一个扩展名和结构来自此。
它们是可以序列化的两种数据结构：映射（如STL映射）和元素序列（如STL向量）（mappings (like the STL map) and element sequence (like the STL vector). ）。它们之间的区别在于：
* 在map中，每个元素都可以通过您可以访问的内容获得唯一的名称。
* 对于序列（sequences ），您需要通过它们来查询特定项目。

##### 1、XML / YAML文件的打开和关闭

XML / YAML文件打开和关闭。在将任何内容写入此类文件之前，您需要打开它并在最后关闭它。OpenCV中的XML / YAML数据结构是[cv::FileStorage](https://docs.opencv.org/3.1.0/da/d56/classcv_1_1FileStorage.html)。要指定这个文件绑定到硬盘上的结构，你可以使用它的构造函数或[open（）](https://docs.opencv.org/3.1.0/d6/dee/group__hdf5.html#gac08f6eafa0f92e1c864044e27bfc7bad)函数：
```C++
string filename = "I.xml";
FileStorage fs(filename, FileStorage::WRITE);
//...
fs.open(filename, FileStorage::READ);
```

您使用第二个参数的其中一个参数是一个常量，指定您可以在其上执行的操作类型：WRITE，READ或APPEND。在文件名中指定的扩展名也决定了将要使用的输出格式。如果您指定了例如* .xml.gz *的扩展名，则输出将会被压缩。

该文件在 ** cv :: FileStorage** 对象被销毁时自动关闭。 但是，您可以通过使用[release ](https://docs.opencv.org/3.1.0/da/d56/classcv_1_1FileStorage.html#ad23d5415a06fb4bc97bfa034589b376e)函数来显式调用它：

```C++
fs.release();                 // explicit close
```
##### 2、输入和输出文字和数字

数据结构使用与STL库相同的 << 输出运算符。为了输出任何类型的数据结构，我们首先需要指定它的名称。我们只需打印出这个名字就可以做到这一点。
对于基本类型和它的值，您可以按照以下打印：
```C++
fs << "iterationNr" << 100;
```
读入是一个简单的寻址(addressing )（通过[]运算符）和铸造(casting )操作或通过>>操作符读取：
```C++
int itNr;
fs["iterationNr"] >> itNr;
itNr = (int) fs["iterationNr"];
```

##### 3、OpenCV数据结构的输入/输出。

这些操作就像基本的C ++类型一样：
```C++
Mat R = Mat_<uchar >::eye  (3, 3),
    T = Mat_<double>::zeros(3, 1);
fs << "R" << R;                                      // Write cv::Mat
fs << "T" << T;
fs["R"] >> R;                                      // Read cv::Mat
fs["T"] >> T;
```


##### 4、矢量（数组）和关联图的输入/输出。

正如我之前提到的，我们也可以输出映射(maps)和序列(sequences)（数组，矢量）。我们再次首先打印变量的名称，然后我们必须指定我们的输出是序列(sequences)还是映射(maps)。
对于第一个元素之前的序列(sequences)，打印“[”字符 并在最后一个之后打印“]”字符：

```C++
fs << "strings" << "[";                              // text - string sequence
fs << "image1.jpg" << "Awesomeness" << "baboon.jpg";
fs << "]";                                           // close sequence
```

对于映射(maps)而言，练习是相同的，但现在我们使用“{”和“}”分隔符：

```C++
fs << "Mapping";                              // text - mapping
fs << "{" << "One" << 1;
fs <<        "Two" << 2 << "}";
```

为了从这些中读取，我们使用[cv :: FileNode](https://docs.opencv.org/3.1.0/de/dd9/classcv_1_1FileNode.html)和[cv :: FileNodeIterator](https://docs.opencv.org/3.1.0/d7/d4e/classcv_1_1FileNodeIterator.html)数据结构。[cv :: FileStorage](https://docs.opencv.org/3.1.0/da/d56/classcv_1_1FileStorage.html)类的[]运算符返回一个[cv::FileNode](https://docs.opencv.org/3.1.0/de/dd9/classcv_1_1FileNode.html)数据类型。如果节点是顺序的，我们可以使用[cv::FileNodeIterator](https://docs.opencv.org/3.1.0/d7/d4e/classcv_1_1FileNodeIterator.html)遍历这些项目：

```C++
FileNode n = fs["strings"];                         // Read string sequence - Get node
if (n.type() != FileNode::SEQ)
{
    cerr << "strings is not a sequence! FAIL" << endl;
    return 1;
}
FileNodeIterator it = n.begin(), it_end = n.end(); // Go through the node
for (; it != it_end; ++it)
    cout << (string)*it << endl;
```

对于映射(maps)，您可以再次使用[]运算符来访问给定项目（或者>>操作符）：

```C++
n = fs["Mapping"];                                // Read mappings from a sequence
cout << "Two  " << (int)(n["Two"]) << "; ";
cout << "One  " << (int)(n["One"]) << endl << endl;
```

##### 5、读写你自己的数据结构。 

###### 假设你有一个数据结构：
```C++
class MyData
{
public:
      MyData() : A(0), X(0), id() {}
public:   // Data Members
   int A;
   double X;
   string id;
};

```

通过在类内部和外部添加读取和写入函数，可以通过OpenCV I / O XML / YAML的接口对它进行序列化（就像OpenCV本身的数据结构一样）。 

###### 对于内部部分：

```C++
void write(FileStorage& fs) const                        //Write serialization for this class
{
  fs << "{" << "A" << A << "X" << X << "id" << id << "}";
}
void read(const FileNode& node)                          //Read serialization for this class
{
  A = (int)node["A"];
  X = (double)node["X"];
  id = (string)node["id"];
}
```

###### 然后你需要在类的外面添加下面的函数定义：

```C++
void write(FileStorage& fs, const std::string&, const MyData& x)
{
x.write(fs);
}
void read(const FileNode& node, MyData& x, const MyData& default_value = MyData())
{
if(node.empty())
    x = default_value;
else
    x.read(node);
}
```

在这里你可以观察到，在read部分，我们定义了如果用户试图读取不存在的节点会发生什么。在这种情况下，我们只返回默认的初始化值，但是更详细的解决方案是返回一个对象ID的负值。一旦添加了这四个函数，就使用>>操作符进行写操作，使用<<操作符进行读操作：

```C++
MyData m(1);
fs << "MyData" << m;                                // your own data structures
fs["MyData"] >> m;                                 // Read your own structure_
```

或者尝试阅读non-existing read：
```C++
fs["NonExisting"] >> m;   // Do not add a fs << "NonExisting" << m command for this to work
cout << endl << "NonExisting = " << endl << m << endl;
```

#### Result:

大多数情况下，我们只是打印出定义的数字。在控制台的屏幕上，您可以看到：
```C++
     Write Done.
     
     Reading:
     100image1.jpg
     Awesomeness
     baboon.jpg
     Two  2; One  1
     
     
     R = [1, 0, 0;
      0, 1, 0;
      0, 0, 1]
    T = [0; 0; 0]
    
    MyData =
    { id = mydata1234, X = 3.14159, A = 97}
    
    Attempt to read NonExisting (should initialize the data structure with its default).
    NonExisting =
    { id = , X = 0, A = 0}
    
    Tip: Open up output.xml with a text editor to see the serialized data.
```

尽管如此，在输出xml文件中可能会看到更有趣的内容：

```C++
     <?xml version="1.0"?>
     <opencv_storage>
     <iterationNr>100</iterationNr>
     <strings>
       image1.jpg Awesomeness baboon.jpg</strings>
     <Mapping>
       <One>1</One>
       <Two>2</Two></Mapping>
     <R type_id="opencv-matrix">
      <rows>3</rows>
      <cols>3</cols>
      <dt>u</dt>
      <data>
        1 0 0 0 1 0 0 0 1</data></R>
    <T type_id="opencv-matrix">
      <rows>3</rows>
      <cols>1</cols>
      <dt>d</dt>
      <data>
        0. 0. 0.</data></T>
    <MyData>
      <A>97</A>
      <X>3.1415926535897931e+000</X>
      <id>mydata1234</id></MyData>
    </opencv_storage>
```
Or the YAML file:

```C++
     %YAML:1.0
     iterationNr: 100
     strings:
        - "image1.jpg"
        - Awesomeness
        - "baboon.jpg"
     Mapping:
        One: 1
        Two: 2
    R: !!opencv-matrix
       rows: 3
       cols: 3
       dt: u
       data: [ 1, 0, 0, 0, 1, 0, 0, 0, 1 ]
    T: !!opencv-matrix
       rows: 3
       cols: 1
       dt: d
       data: [ 0., 0., 0. ]
    MyData:
       A: 97
       X: 3.1415926535897931e+000
       id: mydata1234
```

您可以[在这里](https://youtu.be/A4yqVnByMMM)观看YouTube上的这个运行时实例。

<iframe width="560" height="349" src="https://www.youtube.com/embed/A4yqVnByMMM" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

