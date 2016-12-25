---
title: ogldev-tutorial-00
date: 2016-05-21 10:57:13
categories:
  - opengl
tags:
  - ogldev
---
# 教程介绍
本教程内容来源于http://ogldev.atspace.co.uk/ ，之后将该教程简称为ogldev

本人将逐章节对ogldev进行翻译，希望对大家学习OpenGL有帮助（PS : 我也是边学边翻译，有困难希望大家能多多交流）

# 环境准备
## Fedora
```bash
yum install gcc-c++ glew-devel freeglut-devel ImageMagick-c++-devel ImageMagick-c++ assimp-devel glfw-devel
```
## Ubuntu
```bash
apt-get install g++ freeglut3-dev glew1.5-dev libmagick++-dev libassimp-dev libglfw-dev
ln /usr/lib/pkgconfig/libglfw.pc /usr/lib/pkgconfig/glfw3.pc
```

## Mac
```bash
安装Xcode
brew install assimp
brew install homebrew/x11/freeglut
brew install imagemagick
brew install homebrew/versions/glfw3
brew install glew
brew install fontconfig

ln -s /System/Library/Frameworks/OpenGL.framework/Headers/ /usr/local/include/GL
```

# 教程源码
http://ogldev.atspace.co.uk/ogldev-source.zip
