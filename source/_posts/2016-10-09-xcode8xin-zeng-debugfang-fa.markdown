---
layout: post
title: "Xcode8新增Debug方法"
date: 2016-10-07 23:25
comments: true
categories: iOS
---
今年的WWDC介绍了不少新的调试方法，可以更加快捷便利的定位问题、分析问题、解决问题，下面就简单介绍一下。

##循环布局检测
当你进入某个界面或点击某个按钮后，发现屏幕不再响应事件，然后进入无限等待，debug下可以看到CPU满负荷，RAM也不断增加，那么有可能就进入了循环布局状态。

####举个栗子
先说一个简单的循环布局的case，如下图所示。

<img src="/images/2016/auto_layout_loop.png">

最底下的subview A在layout时更新superview B的`bounds`，B因为不在layout状态，所以会使superview B调用其superview C的`setNeedsLayout()`方法，然后进入layout状态，此时C重置B的`bounds`，B又使A再次layout。因为对B的`bounds`设置不一致，导致循环下去。
<!-- more -->

####检测方法
可在`Launch Arguments`中添加`UIViewLayoutFeedbackLoopDebuggingThreshold`，设置循环布局阀值，如100，然后当循环布局次数超过阀值时会抛出异常，此时通过`po [_UIViewLayoutFeedbackLoopDebugger layoutFeedbackLoopDebugger]`可输出详细信息。

<img src="/images/2016/auto_layout_loop_threshold.png">

##parray poarray
看名字就能猜到，这是针对数组的打印信息命令。在传统的C数组中，是无法通过数组本身知道其长度的，所以一直以来`lldb`都不能很好的打印C数组，`parray`和`poarray`解决了这个痛点，可以通过指定打印个数来打印数组的多个内容。

> 需要注意的是这两个命令都是针对C数组的，区别是以`p`还是`po`的方式打印数组内容。另外只能打印堆空间存储的数组。


``` c
// 以"p"方式打印内容
parray 3 intArray
parray `count` intArray    // "count"为代码中变量
// 以"po"方式打印内容
poarray 3 objectArray
poarray `count` objectArray
```

##register read
`register`命令并不是新增的，一直以来都有，但在WWDC2016`Session 417 Debugging Tips and Tricks`中重点说了一下，实为解决一类疑难bug的利器，在这里也简单介绍下。

平时是否有过crash在没有源码的第三方库或系统库中，而检查自己代码又没有发现任何问题？那么试试`register`命令吧，可能会有意向不到的效果。

通过`register read`可以读取当前状态下寄存器存储的变量，在没有源码的情况下运气好就可以获取到一些非常有用的信息，毕竟很有可能是你传入的某个错误值导致的最终crash，而这个值就可能从寄存器中取到，从而快速定位原因。

`register`还有一个非常有用的特性，当刚进入某一函数时，可通过`register read $arg1 $arg2 ...`读取函数的各个参数值，如下：

<img src="/images/2016/register_read.png">

##启动各过程耗时检测
在WWDC2016`Session 406 Optimizing App Startup Time`中，详细介绍了App启动，即进入`main()`函数前做了什么事儿，以及如何优化各个过程，另外提供了`DYLD_PRINT_STATISTICS`环境变量方便测试各个过程耗时。

<img src="/images/2016/startup_time1.png">

然后在启动时会打印如下所示信息：

<img src="/images/2016/startup_time2.png">

Apple给出的建议是：
> 在最新的设备下，启动时间控制在`400ms`以内

如果你的App不达标，那么就开始着手优化吧！具体各个过程的优化原理及方法，限于篇幅就不再介绍了，感兴趣可以看看[Session 406 Optimizing App Startup Time](https://developer.apple.com/videos/wwdc/2016/?id=406)。

##runtime issue
Xcode8新增了运行时问题的检查，如下：

<img src="/images/2016/runtime_issue.png">

`runtime issue`主要分为3类：
 
 - 线程问题，加入了`Thread Sanitizer`检测线程问题
 - UI布局问题，主要是约束冲突
 - 内存问题，检测内存泄露

##Thread Sanitizer
继`Address Sanitizer`之后，Xcode8中加入`Thread Sanitizer`用于检查一些线程问题，并会显示在`runtime issue`列表中，使用`Thread Sanitizer`需如下图所示开启，重新编译运行后即可。

<img src="/images/2016/thread_sanitizer.png">

`Thread Sanitizer`主要检测以下问题：

 - 使用了未初始化的`mutexes`
 - Thread leaks，如缺少`pthread_join`
 - 在signal handlers中的不安全调用，如调用`malloc`
 - 在错误的线程中做unlock
 - 最重要的一点：`Data races`问题

凡是通过`Thread Sanitizer`检查出来问题，都代表着有很严重的隐患，那么，赶快修复吧！

##Static Analysis
Xcode8对`Static Analysis`做了近一步加强，新增以下问题检测：

 - 本地化检查
 - 内存泄漏检查
 - Nullability检查

####本地化检查
首先需要如下方式开启，`Static Analysis`将会检查`UI元素`的文字设置是否使用了`NSLocalizedString`。

<img src="/images/2016/localizability.png">

####内存泄漏检查
主要检查的是`dealloc`中内存的释放。这个功能来的有点晚，是一个给`MRC`使用的功能，在`ARC`基本普及的今天已经显得不是太重要了，当时没能雪中送炭，如今只能锦上添花了。

<img src="/images/2016/dealloc_release.png">

####Nullability检查
这项检查是针对设置`nonnull`的`property`或`返回值`。当设置为`nonnull`，但返回值可能为`nil`时则会有相应warning。

<img src="/images/2016/nullability.png">

## 参考链接
1. [Section 236 What’s New in Auto Layout](https://developer.apple.com/videos/wwdc/2016/?id=236)
2. [Section 406 Optimizing App Startup Time](https://developer.apple.com/videos/wwdc/2016/?id=406)
3. [Section 410 Visual Debugging with Xcode](https://developer.apple.com/videos/wwdc/2016/?id=410)
4. [Section 412 Thread Sanitizer and Static Analysis](https://developer.apple.com/videos/wwdc/2016/?id=412)
5. [Section 417 Debug Tips and Tricks](https://developer.apple.com/videos/wwdc/2016/?id=417)