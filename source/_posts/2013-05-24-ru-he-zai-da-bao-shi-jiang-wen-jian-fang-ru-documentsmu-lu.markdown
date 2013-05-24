---
layout: post
title: "如何在打包时将文件放入Documents目录"
date: 2013-05-24 14:31
comments: true
categories: iOS
---

今天遇到一个这样的问题，想在打包时将一些文件放进`Documents`目录下，这样用户能在一开始就能有一些文件可用。像类似于一些阅读类app就经常将几本默认书籍放入`Documents`目录下。

首先我们来看一下一个应用程序的文件结构：

<img src="/images/2013/ios_app_layout.jpg">

如图，可以看出`.app`文件和`Documents`文件夹处于同一级，而`.app`文件正是我们build时打包生成的应用程序文件，我们向project中添加的resource文件也当然只处于`.app`中。这样，就想到在`Info.plist`中会不会有这样的设置，可以在应用程序安装时将指定文件添加到`Documents`目录，很可惜，没有这样的设置。

那么，现在看来，要实现这样的功能就只能曲线救国了，思路如下：

将文件随app一起打包，然后在程序第一次运行时将这些文件通过代码copy到`Documents`目录下，当然要记录一个状态位，每次去读取这个状态位来进行判断。

