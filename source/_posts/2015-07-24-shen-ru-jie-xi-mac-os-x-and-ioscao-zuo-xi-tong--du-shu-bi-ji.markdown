---
layout: post
title: "《深入解析Mac OS X &amp; iOS操作系统》读书笔记"
date: 2015-07-24 16:44
comments: true
categories: iOS OSX
---
本书对`OS X`和`iOS`的底层细节讲的非常详细，各方面都有所涉及，对于深入了解`OS X`和`iOS`有很大帮助。对于一般App开发人员来说，我感觉本书内容并不太适合，所以完全以扩充知识面的目标读完本书，以下是以我所关心的内容整理的读书笔记，希望对大家有所帮助。另附[亚马逊购买地址](http://www.amazon.cn/%E6%B7%B1%E5%85%A5%E8%A7%A3%E6%9E%90Mac-OS-X-iOS%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F-%E8%8E%B1%E6%96%87/dp/B00JFJTE86/ref=sr_1_1?s=books&ie=UTF8&qid=1437822172&sr=1-1)方便感兴趣的朋友:)

## 第一章 达尔文主义：OS X的进化史
1. OS X是`Mac OS Classic`和`NeXTSTEP`的融合。
2. `Darwin`是操作系统的类UNIX核心，由kernel、XNU和运行时组成，是OS X和iOS的重要组成部分，OS X的`Darwin`是开源的，除OS X10.0对应Darwin 1.3.x之外，其他版本都符合：`if (OSX.version == 10.x.y) Darwin.version = (4+x).y`
3. 从10.3（Panther）开始，苹果开发了`Safari`替代`IE for Mac`；从10.4.4（Tiger）开始，支持`Intel x86架构`；10.5（Leopard）有了`Objective-C 2.0`；10.6（Snow Leopard）开始完整支持64位，提供`GCD`，完全抛弃`PPC架构`。
4. iOS和OS X对比：
    * iOS基于`ARM`架构，而OS X基于Intel `i386`和`x86_64`。
    * iOS内核代码依然闭源，OS X内核`XNU`则是开源的。
    * iOS内核的编译稍有不同，关注的是嵌入式特性和一些新的API。
    * iOS的系统GUI是`SpringBoard`，OS X为`Aqua`。
    * iOS的内存管理要紧凑得多，因为移动设备没有几乎无穷的交换空间可以使用。
    * iOS应用程序不允许访问底层UNIX API（即Darwin），也没有root访问权限，而且只能访问自己的目录内数据。
<!-- more -->

## 第二章 合众为一：OS X和iOS的架构
1. OS X和iOS层次结构：
    * 用户体验层：包括`Aqua`、`Dashboard`、`Spotlight`和`辅助功能（Accessibility）`，iOS中为`SpringBoard`同时支持`Spotlight`
    * 应用框架层：包括`Cocoa`、`Carbon`和`Java`，而在iOS中只有`Cocoa（Cocoa Touch）`
    * 核心框架：也称为图形和媒体层。包括`核心框架`、`Open GL`和`QuickTime`
    * Darwin：操作系统核心，包括`内核`和`UNIX shell环境`  
    <img src="/images/2015/OSX_iOS_architecture.png" width="50%" height="50%">
2. Darwin架构：  
<img src="/images/2015/darwin_architecture.png" width="50%" height="50%">
3. GUI是由第一个用户态进程`launchd`启动的，支持GUI工作的主进程是`WindowServer`
4. `Carbon`是OS 9遗留编程接口的名称，已废弃
5. XNU包含组件：`Mach微内核`、`BSD层`、`linkern`、`I/O Kit`

## 第三章 站在巨人的肩膀上：OS X和iOS使用的技术
1. `kqueue`是BSD中使用的内核事件通知机制，一个`kqueue`指的是一个描述符，这个描述符会阻塞等待直到一个特定类型和种类的事件发生。如监视文件、Mach port、套接字、发给进程的特定信号、纳秒级定时器、虚拟内存相关通知、vnode相关。
2. `强制访问控制（MAC）`，FreeBSD 5.x最早引入，是OS X`隔离机制（Sandboxing，沙盒机制）`和iOS的`entitlement机制`基础。
3. OS X 10.4（Tiger）引入新的日志模型：`Apple System Log（ASL）`，目标是提供比传统UNIX日志syslog更为灵活的功能。
4. `FSEvents`提供了文件系统通知的API，和Linux的`inotify`类似。
5. 沙盒架构：  
<img src="/images/2015/sandbox_architecture.png">

## 第四章 庖丁解进程：Mach-O格式、进程以及线程内幕
1. UNIX进程生命周期:  
<img src="/images/2015/UNIX_process_life_cycle.png">
2. OS X目前支持三种可执行格式：
    * 解释器脚本格式（以#!后的命令运行脚本，魔数:`#!`）
    * 通用二进制格式（胖二进制格式，魔数：小尾`0xcafebabe`,大尾`0xbebafeca`）
    * Mach-O格式（OS X原生二进制格式，魔数：32位`0xfeedface`,64位`0xfeedfacf`）,系统根据魔数类型加载执行文件。
3. 通用二进制格式本质就是各种架构的二进制文件的打包文件，通过文件头的信息以加载匹配当前架构的二进制文件。
4. OS X上几乎所有的程序都是动态链接的，默认使用`dyld`作为动态链接器，这是一个用户态进程，不属于内核。
5. 32位OS X系统中，用户态和内核态都有完整的4GB地址空间，代价是地址空间切换需要刷新CR3和TLB
6. `__PAGEZERO段`，32位为一个页（4K），64位为4GB。为方便捕获空指针和将整数当做指针引用。
7. OS X中main函数有额外参数apple:
```c
void main (int argc, char **argv, char **envp, char **apple)
```

## 第五章 进程跟踪和调试
1. OS X对`DTrace`支持较为完整，而iOS没有，需使用`CHUD`或`AppleProfileFamily`
2. 获取进程信息系统调用：`sysctl`、`proc_info`
3. 使用未文档化的`stack_snapshot`系统调用，可以捕获指定进程中所有的线程状态。
4. OS X和iOS都没有使用核心转储文件，而是使用`Crash Reporter`生成崩溃日志，用以调试应用程序崩溃。
5. OS X和iOS可将异常端口绑定至BSD进程底层的Mach任务，可很容易实现当应用程序崩溃时自动运行另一个程序。而在UNIX中，很难简单实现，因为只有父进程能收到子进程的死亡通知。
6. 可使用`heap`、`leaks`、`malloc_history`工具调试内存泄漏。

## 第六章 引导过程：EFI和iBoot
1. 大部分PC使用`BIOS`引导（Windows），OS X使用`EFI`引导，iOS使用`iBoot`引导
2. EFI服务提供`引导服务`和`运行时服务`，前者只能只能在EFI模式下使用，后者在退出EFI模式，即操作系统加载后也能使用，I/O Kit重度使用。
3. EFI引导完成后得到最终的`BootStruct`，将其和控制权交给内核完成引导。
4. iOS引导过程：（除引导ROM外，其他步骤都是加密且数字签名的。）  
<img src="/images/2015/iOS_boot_process.png">

## 第七章 launchd
1. OS X和iOS中的`launchd`对应UN*X系统中的`init`，但相对`init`有许多改进。包含了如atd、crond、inetd/xinetd等daemon，支持事务、autorun和文件系统观察，整合I/O Kit。
2. launchd区分两种后台作业：
    * 守护程序（daemon）：和用户没有交互，不考虑是否有用户登录系统。
    * 代理程序（agent）：可以和用户有交互，只有在用户登录时启动。
3. iOS中的`lockdownd`，负责处理设备激活、备份、崩溃报告、设备同步等。
4. OS X中的GUI shell是`Finder`，iOS为`SpringBoard`。
5. `XPC`是Lion和iOS5新引入的轻量级进程间通信原语，目前闭源。

## 第八章 内核架构
1. 内核架构设计类型有`巨内核`（UNIX、Linux）、`微内核`（Mach）、`混合内核`（XNU、Windows）。
2. 内核态、用户态转换分为自愿转换、非自愿转换。
    * 自愿转换：使用内核服务，即系统调用。
    * 非自愿转换：发生异常、中断或处理器陷阱时。
3. XNU中系统调用有4种：`UNIX`、`MACH`、`MDEP`（机器相关调用）、`DIAG`（诊断调用）
4. 32位系统下，UNIX系统调用编号为正数，MACH为负数，64位则都为正数，最高位字节包含调用类型。

## 第九章 由生到死——内核引导和内核崩溃
1. XNU源码中，所有函数的实现都将函数名放在行头，即返回值在上一行，这是为了方便搜索。
2. pid 0是内核进程`kernel_task`（准确说0表示没有pid），`launchd`是第一个用户态进程，pid为1。
3. 可通过`KDP`协议远程调试内核。

## 第十章 Mach原语：一切以消息为媒介
1. Mach的最主要目标就是将所有功能移出内核，放在用户态中，将内核保持在极简状态。
2. Mach同步原语：`互斥体`、`自旋锁`、`信号量`、`锁集`。其中信号量和锁集在用户态下可见。
3. Mach的机器层原语：`主机（host）`、`时钟`、`处理器`和`处理器集的抽象`。

## 第十一章 Mach调度
1. 线程是Mach中最小执行单元，线程是包含在`任务（task）`中的。
2. Mach内核中没有BSD进程概念，而是以任务表示，一个BSD进程对应一个底层Mach任务对象，`kernel_task`即是Mach对于内核的表示。
3. Mach允许创建`远程线程`，即可以在一个任务中创建另一个任务的线程，Windows也可以实现，而UNIX和Linux不支持这种功能。
4. Mach调度优先级有128个，Windows为32个，Linux为140个。其中数字越大，优先级越高。
5. `控制权转交（handoff）`：Mach调度器允许线程主动放弃CPU，并指定某个特定线程运行。
6. `可使用续体（continuation）`：线程可丢弃自己的栈，系统恢复时不需要恢复线程栈，可以明显加快上下文切换速度，此项特性在Mach中应用广泛。
7. 通过`异步软件陷阱（AST）`可使内核响应外带事件，如调度事件，BSD信号基于此实现。
8. `launchd`注册异常端口，其子进程也继承同样端口，`崩溃报告器（crash reporter）`会接受此端口发出的异常，会当发生crash时，崩溃报告器会自动根据需要启动。

## 第十二章 Mach虚拟内存
1. Mach的虚拟内存子系统主要分两层：`虚拟内存层`（机器无关）、`物理内存层`（机器相关）。
2. Mach中`zone`的概念相当于Linux的`memory cache`和Windows的`Poll`。
3. `zone`是一种内存区域，用于快速分配和释放频繁的固定大小的对象。例如，可使用`zprint kalloc`查看`kalloc`的`zone`。
4. Mach的分页器主要有：`Default分页器`、`VNode分页器`、`Device分页器`、`Swapfile分页器`、`Apple-protected分页器`、`Freezer分页器`（iOS）。
5. 分页器只是提供分页操作，不决定具体调度，调度由`pageout`守护线程执行。

## 第十三章 BSD层
1. OS X有`UNIX03`认证，达到源码级兼容，即提供与UNIX统一的API。
2. BSD层在Mach层之上，提供了`POSIX API`。但XNU的BSD不是完整的BSD，即移植部分BSD内容，如`VFS`和`网络架构`。
3. BSD的进程和线程都是在Mach提供原语的基础上进行了封装，BSD进程和线程对应有Mach的任务和线程。
4. XNU中内核线程都是Mach线程，没有对应的BSD线程，同样内核任务`kernel_task`也没有对应的进程（因此其pid为0，表示没有进程pid）。
5. UNIX模型中，进程不能被“创建”出来，只能通过`fork()`系统调用复制出来。`vfork`、`fork`、`posix_spawn`系统调用，底层都是由`fork1()`实现，只是传入参数不同。
6. 除了`DTrace`，XNU在BSD层还提供了其他UNIX具有的`ptrace`，但功能大大缩水，如不能读写其他进程内存。
7. Mach通过`异常机制`处理底层的陷阱，BSD则在`异常机制`之上构建了`信号处理机制`。操作系统和用户产生的信号先被Mach转换为异常，然后再由BSD产生信号。

## 第十四章 有新有旧：BSD高级功能
1. OS X和iOS低内存处理机制称为`Jetsam`，或`Memorystatus`。用于杀掉消耗过多太多内存的进程并抛弃占用内存。
2. iOS中，`Jetsam/Memorystatus`和默认的`freezer`结合使用，实现内存冷冻而不是杀死。
3. 从Mountain Lion和iOS6开始，实行`内核地址空间布局随机化（KASLR）`，以提高系统安全性。
4. `工作队列（work queue）`，作用是为应用程序提供多线程支持并扩展到多处理器支持，为`GCD`提供了基础。
5. MAC是从TrustdBSD引入的强大安全特性，在OS X主要体现在`沙盒机制`，在iOS中主要体现为`entitlement机制`。
6. `sandbox`将所有第三方应用限制为只能访问自己的目录；`AppleMobileFileIntegrity.kext`（用户态守护进程amfid）负责杀掉任何代码签名不正确的进程。

## 第十五章 文件系统和虚拟文件系统交换
1. XUN的`文件系统`是在BSD层实现的，使用了来自Solaris的`VFS框架`（已成为UNIX内核与文件系统实现之间的标准接口）。
2. OS X传统上支持3种分区方案：`主引导记录（MBR）`、`Apple Partition Map（APM）`、`GUID分区表（GPT）`。
3. `MBR`是除OS X和64位Windows之外其他操作系统的默认分区方案，以磁盘第一扇区为引导扇区。
4. `APM`是苹果设计用来取代`MBR`的，现只存于PPC的Mac和iPad Classic和Nano中。
5. 在苹果普遍使用的是`GPT`（包括iOS），属于EFI规范的一部分。
6. `LwVM`是苹果的私有分区方案，继承自GPT，用于iOS5默认分区方案，它允许分区加密。
7. `CoreStorage`是Lion新引入的分区类型，给OS X带来了逻辑卷管理的支持，支持全盘加密，只能创建在`GPT`驱动器上。
8. 所有文件系统都提供了同样的原语，内核对文件的接口称为`虚拟文件系统交换（VFS）`。Mac原生的文件系统为`HFS`（已废弃）、`HFS+`。
9. 即插即用是由守护进程`diskarbitrationd`实现，由launchd启动。
10. `DMG`格式，即磁盘镜像文件，包含了整个文件系统，属于苹果私有格式。从Lion开始，允许指定`DMG`文件用作根文件系统，如安装系统时。

## 第十六章 基于B树的HFS+文件系统
1. DOS原生文件系统为`FAT`，Windows是`NTFS`，Linux是`Ext2/3/4`，OS X为`HFS+`，iOS为`HFSX`。
2. HFS+通过支持`扩展属性`来支持`访问控制表(ACL)`（精确设置任何用户任何组的具体权限）。
3. `扩展属性`可以添加很多额外信息，如文件件的颜色标签、文件下载来源等，可通过`ls -l@`或`xattr`查看，但像文件压缩、ACL则在这些命令中被屏蔽了，需要更底层的方法查看。
4. 系统中有很多文件是通过`HFS+`的压缩属性进行压缩的，如常用的ls命令。可通过ls的-O参数查看。
5. `HFS+`使用的UTF-16编码，文件名最长255个字符。
6. `HFS+`是大小写不敏感的，但保留大小写，而现版本的`HFSX`只是在`HFS+`的基础上变为大小写敏感。
7. `HFS+`使用6个特殊文件维护数据：`编录（catalog）B树`、`属性B树`、`extent溢出B树`、`热文件B树`、`分配文件`、`启动文件`。

## 第十七章 遵守协议：网络协议栈
1. 苹果原本使用自己的`AppleTalk`网络协议栈，后放弃才采用`TCP/IP`。至今仍使用的`Bonjour协议`和`AFP协议`都是`AppleTalk`的遗产。
2. `网络驱动程序套接字(PF_NDRV)`（苹果特有），支持用户态下深入数据链路层直接修改原始数据包。
3. `系统套接字(PF_SYSTEM)`（苹果特有），提供了一种内核空间和用户空间通信的方法。
4. 套接字在内核中是个巨大的数据结构，内核需要维护套接字和文件描述符的映射关系。
5. XNU支持的网络层协议有：`IPv4`、`IPv6`、`AppleTalk`。
6. 在网络接口层，`lo接口`是唯一必须存在的，且属于原生支持接口；`en接口`（以太网或802.11接口）、`fw接口`（IP over FireWire）、`pdp_ip接口`（蜂窝数据连接）、`ppp接口`（Point-to-Point协议）都不是XNU原生支持的，而是通过内核扩展来创建的。
7. `utun`是特殊的原生支持接口，使用的是`系统套接字（PF_SYSTEM）`，`VPN`和其他进程通过这个接口提供一个伪接口，这个伪接口的流量都会重新引导到用户态进程。`utun`发送数据包：  
<img src="/images/2015/utun_send_data.png">

8. XNU支持一下数据包过滤机制，著名的`TCPDump`就是基于`BPF过滤机制`实现的。  
不同数据包过滤机制的比较：  
<img src="/images/2015/packet_filtering_compare.png">

## 第十六章 内核扩展模块
1. OS X和iOS使用`kernelcache`预链接kext，`kernelcache`可以签名、加密（iOS就加密了）。
2. `kernelcache`在OS X下是动态创建的，以加快引导进程；iOS中则是由苹果提供的一个固定的文件，不同iDevice是不一样的。
3. OS X中大部分和kext的接口工作是由守护进程`kextd`完成的，以Mach消息通信，而iOS不存在。
4. 内核内置组件会以`伪kext`的形式出现在kext列表中。

## 第十七章 驱动力——I/O Kit驱动程序框架
1. XNU使用C++开发驱动程序，其运行时环境称为`I/O Kit`，是一套几乎自包含的编程环境，可方便的通过面向对象的特性开发驱动程序。
2. `libkern C++`运行时是`I/O Kit`的基础，定义了所有`I/O Kit`驱动程序都可使用的基础类，如OSObject、OSMetaClass、OSArray、OSDictionary、OSString等。
3. `I/O Kit`维护了一个保存所有对象及对象间关系最新信息的数据库，称之为`I/O Registy`。
4. `I/O Kit`所有驱动程序都是从公共祖先`IOService`继承而来的对象。
5. `I/O Kit`的驱动程序分为两种：`驱动程序（driver）`和`节点（nub）`，节点就是指两个驱动程序之间的适配器，表示被控制的设备。
6. `I/O Kit`驱动程序状态机：  
<img src="/images/2015/IOKit_driver_state_machine.png">

OVER！
