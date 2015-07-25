---
layout: post
title: "iproxy-通过USB使用SSH连接iOS设备"
date: 2013-09-16 20:56
comments: true
categories: iOS 越狱
---
越狱之后用到`SSH`时需要通过`Wi-Fi`来连接，输入命令时反应比较慢，还容易掉线，尤其是在越狱开发时，有时会有砸设备、砸Mac的冲动，当然我砸不起，只是想想。

如果能通过USB连接就好了，既不需要依赖`Wi-Fi`，而且速度非常快，感谢开源社区的大牛们，[usbmuxd](http://cgit.sukimashita.com/usbmuxd.git/)开源库就顺带实现了这个功能。
<!-- more -->
通过[brew](http://brew.sh/)来安装（当然也可以自己去下源码手动安装，由于依赖项比较多，所以很繁琐）
``` sh
brew install usbmuxd
```
安装`usbmuxd`库之后，就顺带安装了一个小工具`iproxy`，该工具会将设备上的端口号映射到电脑上的某一个端口，例如：
``` sh
iproxy 2222 22
```
以上命令就是把当前连接设备的22端口(SSH端口)映射到电脑的2222端口，那么想和设备22端口通信，直接和本地的2222端口通信就可以了。
因此，SSH连接设备就可以这样连接了：
``` sh
ssh -p 2222 root@127.0.0.1
```
这样就再也不用依赖`Wi-Fi`了，而且反应很流畅，当然此工具不仅可以用于SSH，也可以映射其他端口，这个就看个人需求了。
