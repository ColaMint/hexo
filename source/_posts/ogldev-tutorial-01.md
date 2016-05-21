---
title: ogldev-tutorial-01
date: 2016-05-21 10:59:31
categories:
- opengl
tags:
- ogldev
---
# Tutorial01
## 背景
OpenGL没有特定的API用来创建和操作窗体。现今支持OpenGL的窗体系统中都有一套子系统来让窗体系统与OpenGL上下文绑定：X窗体有GLX，Windows系统有WGL，MacOS有CGL。如果在不同的窗体系统中编程要用到不同的窗体API，就太麻烦了，所有我们使用一个更高层次的库来提供统一的窗体API。这个库就是“`OpenGL utility library`”，简称 `GLUT`。GLUT提供了一套简单的API来进行窗体管理、事件处理、IO控制和一些其他工作。由于GLUT是跨平台的，所以移植程序很方便。GLUT的替代品包括`SDL`和`GLFW`。

## 程序解读
```c++
#include <GL/freeglut.h>

static void RenderSceneCB()
{
    glClear(GL_COLOR_BUFFER_BIT);
    glutSwapBuffers();
}

static void InitializeGlutCallbacks()
{
    glutDisplayFunc(RenderSceneCB);
}


int main(int argc, char** argv)
{
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_DOUBLE|GLUT_RGBA);
    glutInitWindowSize(1024, 768);
    glutInitWindowPosition(100, 100);
    glutCreateWindow("Tutorial 01");

    InitializeGlutCallbacks();

    glClearColor(0.0f, 0.0f, 0.0f, 0.0f);

    glutMainLoop();

    return 0;
}
```

```c+++
glutInit(&argc, argv);
```
这个调用初始化GLUT，并且可以从命令行获取参数，例如：
`-sync`：取消X窗体异步的特性
`-gldebug`：自动检查GL错误并打印出来

```c++
glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGBA);
```
这里我们配置了一些GLUT选项。`GLUT_DOUBLE`开启双缓冲模式(一个缓冲区用于显示，一个缓冲区用于绘图)。`GLUT_RGBA`指定窗口的颜色模式为RBGA。

```c++
glutInitWindowSize(1024, 768); 
```
指定窗口的大小为1024*768。

```c++
glutInitWindowPosition(100, 100); 
```
指定窗口的起始坐标为(100, 100)。

```c++
glutCreateWindow("Tutorial 01");
```
指定窗口的标题为"Tutorial 01"。

```c++
glutDisplayFunc(RenderSceneCB);
```
程序中与窗体的交互是通过事件回调函数来实现的。GLUT提供了一些回调的选择，但这里我们只用了绘图回调来绘制窗体的内容。我们所注册的这个绘图回调函数会被GLUT的内部循环不断调用。

```c++
glClearColor(0.0f, 0.0f, 0.0f, 0.0f);
```
这里是我们第一次遇到OpenGL中关于`状态`的概念。绘图是一个很复杂的过程，如果每个绘图函数都要接收一大堆参数的话，编程就会很困难了。况且，你经常会想要在几个连续的绘图操作中保持同样的配置。因此，OpenGL中的配置被设计成是有`状态`的。当调用一个修改配置的函数后，这个修改的效果会持续到你下次调用函数来替换这个配置项的值。通过调用上面这个函数，我们就指定了我们在清除帧缓冲区时所用的颜色。这个颜色有四个通道（RGBA），每个通道的值被归一化到在0.0~1.0之间。

```c++
glutMainLoop();
```
调用这个函数之后，我们就把程序的控制权交给了GLUT，它会开始一个内部的循环。在这个循环中，它会监听窗体系统的事件，并通过我们设置的回调函数将事件传递给我们。我们本程序中，GLUT只会调用我们注册的绘图函数（RenderSceneCB）来给我们机会去绘制帧的内容。

```c++
glClear(GL_COLOR_BUFFER_BIT); 
glutSwapBuffers();
```
在我们的绘图函数中，我们做的唯一一件事就是利用我们设定好的颜色来清除帧缓冲区。第二个函数调用让GLUT交换前后缓冲区的角色。

## 程序运行效果
![tutorial01](/static/ogldev/tutorial01.png)


## 帮助
使用OSX的同学可使用如下命令来编译本程序
```bash
g++ -framework OpenGL -framework GLUT -I../Include -DGL_DO_NOT_WARN_IF_MULTI_GL_VERSION_HEADERS_INCLUDED -o tutorial01 tutorial01.cpp
```
