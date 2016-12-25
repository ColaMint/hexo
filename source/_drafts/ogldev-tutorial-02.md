---
title: ogldev-tutorial-02
date: 2016-05-21 15:37:46
categories:
  - opengl
tags:
  - ogldev
---
# Tutorial02

## 原文地址
http://ogldev.atspace.co.uk/www/tutorial02/tutorial02.html

## 背景
这是我们第一次接触`GLEW`，它是一个OpenGL扩展库，能帮助你解决管理OpenGL扩展的头疼问题。GLEW在初始化的时候会自动搜索你所使用的平台上的可用OpenGL扩展，并自动地加载它们。

在本节教程中。我们会第一次看到`顶点缓冲区对象`（VBO，vertext buffer object）的使用方法。从它的名字我们就能看出它是用来保存顶点的。在3D世界中的物体，像怪物、城堡或者是一个简单的旋转着的小方块，都是通过连接一组顶点来构建的。使用顶点缓冲区对象是将顶点载入GPU的最有效的方式。

本节和下节教程是这个系列教程中唯一两节依赖`固定流水线`，而不是`可编程流水线`的教程。之后的教程我们将会对流水线进行彻底的学习。不过就目前来说，能理解如下概念就足够了。在到达`光栅化阶段`(rasterizer，即使用屏幕坐标进行画点、线和三角形的阶段)之前，可见的顶点有X, Y, Z三个坐标，范围在[-1.0,1.0]。光栅化阶段将这些坐标映射到屏幕空间（例如，如果屏幕的宽度是1024，那么X轴的-1.0被映射到0，X轴的1.0被映射到1023）。最后，光栅化程序根据绘图调用中指定的拓扑来绘制图元。因为我们没有给流水线绑定任何`着色器`(shader)，所以我们的顶点没有经历任何变化。这也就意味着我们想让顶点可见，只需要赋予顶点在[-1.0,1.0]间的值即可。如果令X和Y都为0，那么顶点就被放置在两个轴的中点，也就是屏幕的中心。

补充说明：图形数据是以图形为对象形式的表示。图形对象是指`图元`（primitive）和`图段`（segment）。图元有点、线、面、字符、符号、像元阵列等。图段是由图元组成，例如房子中的门、窗。

安装GLEW的方法请参考`ogldev-tutorial-00`。

## 程序解读
```c++
#include <stdio.h>
#include <GL/glew.h>
#include <GL/freeglut.h>
#include "ogldev_math_3d.h"

GLuint VBO;

static void RenderSceneCB()
{
    glClear(GL_COLOR_BUFFER_BIT);

    glEnableVertexAttribArray(0);
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, 0);

    glDrawArrays(GL_POINTS, 0, 1);

    glDisableVertexAttribArray(0);

    glutSwapBuffers();
}


static void InitializeGlutCallbacks()
{
    glutDisplayFunc(RenderSceneCB);
}

static void CreateVertexBuffer()
{
    Vector3f Vertices[1];
    Vertices[0] = Vector3f(0.0f, 0.0f, 0.0f);

    glGenBuffers(1, &VBO);
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(Vertices), Vertices, GL_STATIC_DRAW);
}


int main(int argc, char** argv)
{
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_DOUBLE|GLUT_RGBA);
    glutInitWindowSize(1024, 768);
    glutInitWindowPosition(100, 100);
    glutCreateWindow("Tutorial 02");

    InitializeGlutCallbacks();

    // Must be done after glut is initialized!
    GLenum res = glewInit();
    if (res != GLEW_OK) {
      fprintf(stderr, "Error: '%s'\n", glewGetErrorString(res));
      return 1;
    }

    glClearColor(0.0f, 0.0f, 0.0f, 0.0f);

    CreateVertexBuffer();

    glutMainLoop();

    return 0;
}
```

```c++
#include <GL/glew.h>
```
这里我们引入了GLEW的头文件。需要注意的是，必须在引入其他OpenGL相关的头文件引入此头文件，否则GLEW可能会报错。为了让你的程序链接上GLEW，需要构建的时候加上`-lGLEW`。

```c++
#include "ogldev_math_3d.h"
```
在本节教程中我们会开始使用一些结构体，例如顶点。随着教程的进行，我们会逐步对这个头文件展开讲解。

```c++
GLenum res = glewInit();
if (res != GLEW_OK) {
  fprintf(stderr, "Error: '%s'\n", glewGetErrorString(res));
  return 1;
}
```
这里我们初始化GLEW并检查是否有错误。这步必须在GLUT初始化后完成。

```c++
Vector3f Vertices[1];
Vertices[0] = Vector3f(0.0f, 0.0f, 0.0f);
```
我们建立一个大小为1，类型为Vector3f（这个类型在ogldev_math_3d.h中定义）的数组，并把数组中唯一的元素的XYZ都初始化为0，这将让这个点在显示在屏幕的中央。

```c++
GLuint VBO;
```
我们定义了一个类型为GLuint的全局变量来保存顶点缓冲区对象的`句柄`（handle）。之后你会发现大多数OepnGL对象都是通过一个类型为GLunit的变量来访问的。

```c++
glGenBuffers(1, &VBO);
```
OpenGL定义了几个`glGen*`的函数来生成不同类型的对象，这些对象由OpenGL负责分配和回收。它们通常需要传入两个参数 - 第一个参数指定了想要创建的对象的数目，第二个参数是一个用来保存所分配的对象的句柄数组，数组元素的类型为GLunit。后续对这些函数的调用不会再生成之前已经返回的对象的句柄，除非你先调用`glDeleteBuffers`来删除它们（即回收）。需要注意的是，这里你没有指定你想要拿生成的缓冲区来做什么，因此它们可以被认为是通用的。缓冲区的用途是经过来下面这个函数来指定的。

```c++
glBindBuffer(GL_ARRAY_BUFFER, VBO);
```
OpenGL有一个与众不同的使用句柄的方式。在众多API中句柄只是被简单地作为参数传入相关函数，然后函数对句柄进行操作。在OpenGL中我们把句柄与一个`目标`（target）绑定起来，然后在那个目标上执行命令。这些命令会影响被绑定的句柄直到有另外的句柄代替绑定在该目标上，或者在上面这个函数中的句柄参数传入0来取消绑定。目标`GL_ARRAY_BUFFER`的意思是，与之绑定的缓冲区是一个顶点数组。另外一个有用的目标是`GL_ELEMENT_ARRAY_BUFFER`，它的意思是与之绑定的缓冲区是另外一个缓冲区的索引数组。还有其他的目标我们将在之后的教程中看到。

```c++
glBufferData(GL_ARRAY_BUFFER, sizeof(Vertices), Vertices, GL_STATIC_DRAW);
```
在绑定我们的对象之后，我们要填充它们的数据。上面这个函数第一个参数传入的是要操作的目标，第二个参数是所填充的数据的字节数，第三个参数是顶点数组的地址，第四个参数是一个标志，意味着这个数据的使用模式。由于我们之后不会改变缓冲区的内容，这里我们制定第四个参数为`GL_STATIC_DRAW`。与`GL_STATIC_DRAW`相反的标志是`GL_DYNAMIC_DRAW`。虽然这个标志只是对OpenGL的一个优化提示，但我们还是要深思熟虑之后选择正确的标志来使用。OpenGL驱动会将依赖这个值进行优化（例如内存中哪部分是保存这个缓冲区的最好位置）。

```c++
glEnableVertexAttribArray(0);
```
在着色器教程中你将会看到在着色器中使用的顶点属性有一个映射到它们的索引，这些索引让你能够将程序中的数据与着色器中的属性名称绑定在一起。另外，你必须激活每个顶点的属性索引。在本节教程中，我们不会使用任何着色器，除了我们已经载入缓冲区的顶点位置，所以我们使用在固定流水线中的顶点属性索引0（这在没有着色器绑定时有效）。（这段翻译的好别扭）

```c++
glBindBuffer(GL_ARRAY_BUFFER, VBO);
```
这里我们又将我们的缓冲区绑定了一次，为我们的画图调用做准备。在这个小程序中我们只用到一个顶点缓冲区，所以做在画每一帧时做这个调用是多余。但是在更复杂的程序中，里面会有很多个缓冲区来保存不同的模型，那时你就必须重新绑定缓冲区来更新流水线的状态。

```c++
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, 0);
```
这个调用告诉流水线如何解释缓冲区中的数据。第一个参数指定属性的索引，在本例子中我们知道它默认是0，但是当我们使用着色器时，我们要么需要明确地设置着色器中的索引，要么查询它。第二个参数是属性的组成成分的数目（3代表X, Y, Z）。第三个参数是每个组成成分的类型。第四个参数指定是否在流水线中使用之前将属性值规范化。在本例子中，我们不想要我们的数据在在传输过程中被改变，所以使用`GL_FALSE`。第五个参数（称为`stride`，即步进）是缓冲区中相邻的两个属性实例之间间隔的字节数。由于这里只有一个属性（缓冲区中只有一个顶点的位置），并且数据是紧凑排列的，所以我们传入0。如果我们有一个结构体数组，每个结构体包含位置和法线（由3个浮点数表示的向量），那么我们应该在这个参数传入结构体的大小，即6 * 4 = 24字节。第六个参数在我们刚刚提到的这个例子中很有用。当流水线在寻找我们的属性时我们需要指定属性在结构体中的偏移量。在刚刚这个例子中，位置的偏移量是0，法线的偏移量是12。

```c++
glDrawArrays(GL_POINTS, 0, 1);
```
最后，我们调用这个函数来画几何图形。在这之前的所有命令都很重要，但它们只是设置了我们的画图命令的状态。这时GPU才真正开始工作。
它将结合绘图函数的参数和在此之前为这个点所设置的状态，然后将结果渲染在屏幕上。

OpenGL提供了几种画图调用，每一种适合于不同的场合。一般情况下，你可以将它们分为两类 - `有序绘图`（ordered draws）和`索引绘图`（indexed draws）。有序绘图比较简单，GPU遍历你的顶点缓冲区，将根据绘图调用中指定的拓扑来解释这些顶点。举个例子，如果你指定拓扑为`GL_TRIANGLES`，那么第0~2的顶点将构成第一个三角形，第3~5的顶点将构成第二个三角形，以此类推。如果你想要同一个顶点不止出现在一个三角形中，那么你需要在顶点缓冲区中存储该顶点多次，这将很浪费空间。

索引绘图比较复杂，并且需要额外一个缓冲区，称为`索引缓冲区`（index buffer）。索引缓冲区包含着顶点缓冲区中顶点的索引。GPU扫描索引缓冲区，并让第0~2的索引在顶点缓冲区中所对应的顶点构成第一个三角形，以此类推。如果你想要同一个顶点在出现在两个三角形中，那么只要让该索引在索引缓冲区中出现两次即可。顶点缓冲区值只需要保存每个顶点的一份数据。索引绘图在游戏编程中更常用，因为大多数模型是从代表平面的三角形中创建出来的（例如人的皮肤，城堡的墙壁等等），并且模型中的顶点被多个平面使用。

在本节教程中我们使用了最简单的绘图调用 - `glDrawArrays`，这是一种有序绘图，所以没有用到索引缓冲区。我们指定拓扑类型为`GL_POINTS`，这意味每一个顶点是一个点。第二个参数是所要画的第一个顶点的索引。在本例子我们从缓冲区的第一个顶点开始画，所以传入0。当我们在一个缓冲区中存入多个模型的顶点，这时就可以利用这个参数来选择对应模型的第一个顶点开始绘图了。最后一个参数指定参数绘图的顶点数目。

```c++
glDisableVertexAttribArray(0);
```
在顶点属性使用完之后，使顶点属性无效是一种好习惯。当着色器没有被使用，却保留顶点属性有效时，等于自找麻烦。（不太懂）

本节中与`VertexAttrib`相关的函数看不懂暂时放下吧，因为我也看不懂...

## 程序运行效果
![tutorial02](/static/ogldev/tutorial02.png)


## 帮助
使用OSX的同学可使用如下命令来编译本程序
```bash
g++ -framework OpenGL -framework GLUT `pkg-config --cflags --libs ImageMagick++ assimp glew freetype2 fontconfig` -I../Include -DGL_DO_NOT_WARN_IF_MULTI_GL_VERSION_HEADERS_INCLUDED -o tutorial02 tutorial02.cpp
```
