---
title: Raspberry Pi 电子相册
tags:
  - album
categories:
  - raspberry-pi
date: 2016-10-15 22:41:30
---

# github项目
[https://github.com/Everley1993/QQAlbumPi](https://github.com/Everley1993/QQAlbumPi)

# 使用方法
```
# 拉取代码
git clone https://github.com/Everley1993/QQAlbumPi
cd QQAlbumPi

# 安装依赖库
pip install -r requirement.txt

# 获取公开相册列表（关键是获取相册ID）
# qq: QQ号
python list_album.py --qq {qq}

# 运行服务端
# port: 服务端监听的端口号，默认为2333
# qq: QQ号
# albumid: 相册ID
# interval: 切换图片的时间间隔，单位秒，默认10秒
# display_mode： 图片播放模式，order: 顺序播放，shuffle: 随机播放，默认为shuffle
python server.py --port {port} --qq {qq} --albumid {albumid} --interval {interval} --display_mode {display_mode}

# 浏览器访问http://localhost:{port}，并开启全屏模式
```
# 效果
![](/static/raspberry-pi-album/p1.JPG)
![](/static/raspberry-pi-album/p2.JPG)
