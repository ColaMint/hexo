---
title: Raspberry Pi安装PepperFlash，让Chromium支持Flash
tags:
  - flash PepperFlash
categories:
  - raspberry-pi
date: 2016-10-15 11:42:29
---
```
# 在http://mirror.archlinuxarm.org/armv7h/alarm/上找PepperFlash的下载地址

wget http://sg2.mirror.archlinuxarm.org/armv7h/alarm/chromium-pepper-flash-12.0.0.77-1-armv7h.pkg.tar.xz

tar -xvf chromium-pepper-flash-12.0.0.77-1-armv7h.pkg.tar.xz

cp usr/lib/PepperFlash/libpepflashplayer.so /usr/lib/chromium-browser

# 重启Chromium
```
