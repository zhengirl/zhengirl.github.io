---
title: 树莓派4B编译安装OpenCV4
date: 2019年10月17日 10:10:10
categories: 树莓派
tags: [树莓派4B,OpenCV4,环境搭建]
---

## 树莓派4B编译安装OpenCV4

### 一、找到官方的安装教程

​	这个教程是谷歌出来的，树莓派4B是官方推出的最新版，在树莓派3B上面的安装成功的教程不适用于树莓派4B，会遇到各种难以解决的报错问题，所以找一个好教程非常重要。教程地址：https://qengineering.eu/install-opencv-4.1-on-raspberry-pi-4.html。不科学上网也是可以打开的，这里我就不翻译了。推荐把SD卡的内存升级一下，16G的内存卡就会很勉强，需要删除一些不经常使用的软件，为了避免因内存不足出现的错误，建议直接将内存卡升级到32G或64G。

<!--more-->

​	在安装OpenCV的依赖过程中，我有两个依赖安装失败了，

```bash
sudo apt-get install libgtk2.0-dev libcanberra-gtk*
sudo apt-get install gcc-arm*
```

​	搜了一下也没有找出合适的解决方案，就没有管这个了，但是似乎不影响后面的编译。

### 二、判断是否安装成功

在命令行中输入`python3`，然后输入`import cv2`，如果没有报错就说明成功安装，输入`cv2.__version__`查看cv2的安装版本，成功退出即可。python3是非常容易调用OpenCV的，但是如果想要用C++来调用，相对而言就复杂多了。

### 三、读取照片并显示

​	OpenCV3和OpenCV4版本不兼容，所以如果你直接把OpenCV3运行在OpenCV4上是会报错的。我就吃了这个亏，刚开始也不是特别熟悉OpenCV，所以走一些弯路是难免的。编译C++文件有两种方式，cmake和g++，这里推荐使用cmake，更稳定。在移植代码的时候不容易出错。

参考网址：https://www.aiuai.cn/aifarm792.html

新建`opencv_test.cpp`文件

```C++
#include <iostream>
#include <opencv2/opencv.hpp>

using namespace cv;
using namespace std;

int main(int argc, char** argv)
{

    if ( argc != 2 )
    {
        cout << "Need to load an image ..." << endl;
        return -1;
    }
    Mat image;
    image = imread( argv[1], 1 );
    if ( image.empty() )
    {
        cout<<"No image data!"<< endl;
        return -1;
    }
    namedWindow("Display Image", WINDOW_AUTOSIZE );
    imshow("Display Image", image);
    waitKey(0);
    return 0;
}
```

新建`CMakeLists.txt`文件：

```C
// opencv4 需要 c++ 11 支持
set(CMAKE_CXX_FLAGS "-std=c++11")  
cmake_minimum_required(VERSION 2.8)
  // 工程名
project( DisplayImage )
// 下面的DIR为opencv4的安装位置
set(OpenCV_DIR /usr/local/lib/cmake/opencv4)
find_package(OpenCV REQUIRED )
  // 这一排代表要编译的文件
add_executable( opencv_test opencv_test.cpp )
target_link_libraries( opencv_test ${OpenCV_LIBS} )
```

当前路径下存放待显示的图片`test.jpg`，cmake文件中需要更改自己的OpenCV4的安装目录。执行

```bash
cmake . 
make
./opencv_test test.jpg
```

显示图片。

### 四、读取摄像头数据并显示

打开摄像头并显示成功。同样是采用cmake的方式编译，套路和上面一样。

```c++
#include <opencv2/opencv.hpp>
using namespace cv;
using namespace std;
 
int main()
{
	//读取视频或摄像头
	VideoCapture capture(0);
 
	while (true)
	{
		Mat frame;
		capture >> frame;
		imshow("读取视频", frame);
		waitKey(30);	//延时30
	}
	return 0;
}
```

### 五、总结

1. 找个靠谱的安装opencv的教程比什么都重要，这里再次点赞这个博客。教程地址https://qengineering.eu/install-opencv-4.1-on-raspberry-pi-4.html，但是里面有两个依赖安装失败，尾部含有`*`号，`libcanberra-gtk*`和`gcc-arm*`，但是似乎不影响后面的编译运行。
2. 经过了一夜的编译，第二天上午一看，编译成功了。
3. 前一天在树莓派3B上成功运行了opencv3的程序，采用g++ 方式编译运行，在树莓派4B上希望用同样的代码跑通，但是各种报错。主要原因有两个：opencv最好用cmake的方式编译，用g++本来就是很不稳妥的；其次，opencv3和opencv4的版本兼容问题。知乎上一篇文章，[opencv4从入门到放弃](https://zhuanlan.zhihu.com/p/54845295)，本来被opencv2的库找不到苦恼了一上午，以为是救星，这篇文章说先试着运行samples里面的例子，但是我cmake编译时报错，现在想想可能是OpenCV4的安装路径没有匹配。首先是cpp代码中OpenCV4的版本问题，其次`CMakeLists`中第一行的代码出错；于是我就开始谷歌找错的过程，希望能够解决。均无果。愈加烦躁。各种努力，仍旧报错。
4. 于是只好放弃官方示例中的程序，自己去找了一篇通过opencv显示图片的代码，自己修改CMakeLists的内容，于是竟然跑通了。注意：ssh中执行这个代码是无法成功的，可以编译通过，但是必须最终用树莓派自身执行。
5. 继续尝试寻找显示摄像头的的代码，前面的铺垫工作做好，把CMakeLists略微修改一下即可。
6. 花了将近一天加一个晚上终于调试通过。

