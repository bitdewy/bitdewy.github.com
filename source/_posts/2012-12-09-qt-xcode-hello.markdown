---
layout: post
title: "Xcode 配置 Qt 开发环境手记"
date: 2012-12-09 13:10
comments: true
categories: [Qt, Xcode]
---

首先，这是一个奇葩的需求，不要问为什么不用 Qt Creator，为什么不用 qmake + make，没有那么多为什么。

其次，准备好 Xcode 先。正文开始：

##安装Qt library

Qt5 已经发布 RC1 版本，不过还没到不影响正常使用的程度，所以还是先用 4.8.4，下载链接点[这里][00]，MAC 版的 release 和 debug 库是分开的，可以自由选择。一路安装，下一步就 OK。

  [00]: http://qt-project.org/downloads


<!-- more -->
##生成 xcodeproj

生成 xcodeproj 文件(夹)，(它是文件还是文件夹？)，先创建目录，然后进入，使用 `qmake -project` 生成 pro 文件，如果找不到 `qmake`，那就需要手工找，或者重新安装 Qt library 了，生成 pro 文件之后，就可以用 `qmake -spec macx-xcode` 生成 xcodeproj 了，然后，就没有然后了，直接用 Xcode 打开就 OK。

``` sh
localhost:~ bitdewy$ mkdir Projects/qt_xcode_hello  #建个文件夹先
localhost:~ bitdewy$ cd Projects/qt_xcode_hello/    #进入新建的文件夹
localhost:qt_xcode_hello bitdewy$ qmake -project    #生成pro文件 
localhost:qt_xcode_hello bitdewy$ ls                #看看生成了啥玩意儿
qt_xcode_hello.pro
localhost:qt_xcode_hello bitdewy$ qmake -spec macx-xcode  #生成xcodeproj
localhost:qt_xcode_hello bitdewy$ ls                      #再看看
qt_xcode_hello.pro		qt_xcode_hello.xcodeproj
localhost:qt_xcode_hello bitdewy$ open qt_xcode_hello.xcodeproj/  #用 Xcode 打开
```


##配置 Xcode 工程

打开个空工程，自然是毛也没有，首先工程中新建个文件，然后新建 target 选 other -> External Build System ，Product Name 就叫 qmake，Build Tool 指定到 qmake 的路径，Finish 之后，删除 Arguments 中的 `$(ACTION)`。Command + D，Duplicate 出来个 qmake copy，改名 `qmake -project`，Arguments 填上 `-project`，先run `qmake -project`，再run `qmake`，xcode 工程就和谐了，现在可以写代码跑了。

明白人一看就知道上面是在干什么了，新建文件，然后 `qmake -project` 更新pro文件，再 `qmake` 生成 makefile。整个过程实际上就三行 shell 搞定。

``` sh
localhost:qt_xcode_hello bitdewy$ touch qt_xcode_hello.cpp
localhost:qt_xcode_hello bitdewy$ qmake -project
localhost:qt_xcode_hello bitdewy$ qmake
```


##为毛不直接生成能用 Xcode 工程

更明白的人可能要说脏话了，你妹的搞这么麻烦，生成 xcodeproj 之前补上一句 `touch qt_xcode_hello.cpp` 不就不用配置 XCode 工程了么？ bitdewy 也不是二货，为毛要配置 Xcode 工程？ 因为 Qt 的 Meta Object Compile，用到 signal 和 slot 的时候，都需要 qmake 生成 moc 文件，所以先配置一下没什么坏处这样。


####参考

1. [Qt4 with Xcode](http://qtnode.net/wiki/Qt4_with_Xcode)

