---
layout: post
title: "xCode下自动打包为ipa文件"
date: 2013-07-05 12:58
comments: true
categories: iOS
---

最近经常给测试发包，用老办法的话总是先生成app文件，然后拖到iTunes下生成ipa文件，虽然说过程简单，但重复做这么件事总会觉得麻烦。因此用xCode命令行工具提供的`xcrun`工具写成shell，然后再添加到xCode的工程下，这样就很方便的在每次build之后就能生成相应的ipa文件。shell如下：
``` sh
#!/bin/sh

rm -rdf build
mkdir build
exec >& build/shell.log

/usr/bin/xcrun -sdk iphoneos PackageApplication -v "$BUILT_PRODUCTS_DIR/$PRODUCT_NAME.app" -o "$SRCROOT/build/$PRODUCT_NAME.ipa"
```
会在工程根目录下生成一个build文件夹，然后会把生成的日志和ipa文件放到文件夹下。
然后在project下的`Build Phase`下`Add Run Script`将shell路径添加进去。
<img src="/images/2013/add_script_in_xcode.png">

