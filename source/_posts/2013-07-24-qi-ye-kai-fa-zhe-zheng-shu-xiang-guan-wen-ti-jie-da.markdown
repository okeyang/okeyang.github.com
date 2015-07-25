---
layout: post
title: "企业开发者证书(In-House Distribution)相关问题解答"
date: 2013-07-24 16:00
comments: true
categories: iOS
---
<img src="/images/2013/ios_developer_enterprise_program.png">
最近对企业开发者证书（299刀的）产生了兴趣，查找发现相关资料很少，因此整理了一下相关资料，希望对有同样问题的朋友有所帮助。
<!-- more -->
## 1、是否有分发设备数目限制？

如果使用`Ad-hoc`证书是有限制的，100个设备，但用`In-House Distribution`证书就不存限制了，可以说它是iOS中权限最大的证书（不仅没有分发数目限制，而且可以通过多种途径分发app，更重要的是不需要进行应用程序审核）。

## 2、是否需要绑定UDID？

`Ad-hoc`需要，毕竟只是用于测试目的的。`In-House Distribution`不需要。

## 3、企业开发者证书能在App store发布应用程序吗？

不能，需要另外购买99刀的开发者证书。

## 4、In-House Distribution证书如何分发app？

- 将应用程序分发给用户以使用`iTunes`进行安装。
- 让IT管理员使用`iPhone 配置实用工具`或`Apple Configurator`将应用程序安装在设备上。
- 将应用程序发布到安全 Web 服务器；用户以无线方式访问和执行安装（OTA）。
- 使用MDM服务器来指示受管设备安装内部应用程序或 App Store 应用程序（如果MDM服务器支持该功能）。

## 5、In-House Distribution证书泄露怎办？

如果泄露，可能导致外部人员使用企业app，此时可以撤销证书，然后生成新证书重新分发app。这样旧证书分发的app就无法使用了。原理是这样的：用户首次打开应用程序时，会通过联系 Apple 的 OCSP 服务器来验证分发证书，OCSP 响应会在设备上缓存一段时间（OCSP 服务器所指定的时间段），一般为 3 天到 7 天之间。在此期间将不会再次检查证书的有效性，直至设备重新启动且缓存的响应过期为止。如果那时收到撤销命令，则应用程序将被阻止运行。

## 6、能使用In-House Distribution随意分发app，建立类似第三方App store吗？

理论上可以，但是这是违反Apple协议的，这是要负法律责任的。一般能申请得到企业开发者证书的都是500人以上具有一定规模的企业，这样的企业不会明目张胆去违反Apple的协议。再说，在上一个问题中说明的证书验证原理表明，Apple有能力监控到一个`In-House Distribution`分发的app的用户使用情况，当发现有明显异常（随意分发的用户人数肯定远大于公司员工人数），相信Apple不会坐视不管。话又说回来，只是少量的分发给一些客户或个人的话，这也没有什么大碍。因此这完全取决于随意分发的量。

## 参考链接

1. [Distributing Enterprise Apps for iOS Devices](http://developer.apple.com/library/ios/#featuredarticles/FA_Wireless_Enterprise_App_Distribution/Introduction/Introduction.html)

2. [Has anyone had experience with the iOS Developer Enterprise program?](http://www.linkedin.com/groups/Has-anyone-had-experience-iOS-72283.S.52864455)
