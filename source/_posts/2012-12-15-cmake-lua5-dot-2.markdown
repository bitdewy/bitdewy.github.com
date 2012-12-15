---
layout: post
title: "CMake lua 5.2"
date: 2012-12-15 18:09
comments: true
categories: [cmake, lua]
---

不知道从什么时候开始，lua 源码不再提供 visual studio 工程文件了。windows 用户只能自己动手了，无奈 visual studio 版本太多，工程文件也不是人能读懂，简直不可维护。所以下面用 [CMake][] 来生成 vcproj，这样维护起来更方便。

##什么是 CMake？

[CMake][] 是个开源的跨平台自动化建构系统，它用组态档控制建构过程（build process）的方式和 Unix 的 Make 相似，只是 [CMake][] 的组态档取名为 CakeLists.txt。[CMake] 并不直接建构出最终的软件，而是产生标准的建构档（如 Unix 的 Makefile 或 Windows Visual C++ 的 projects/workspaces），然后再依一般的建构方式使用。这使得熟悉某个集成开发环境（IDE）的开发者可以用标准的方式建构他的软件，这种可以使用各平台的原生建构系统的能力是 [CMake][] 和 SCons 等其他类似系统的区别之处。

有了 [CMake][] 就不用在 vcproj 的各个版本之前来回的切换了，每次用 [CMake][] 生成相应的工程文件就一切 OK 了。

  [CMake]: http://www.cmake.org/


##Windows 下用 CMake 安装 lua 5.2.1

###安装 CMake

从[官方下载][00]相应的安装包，安装完成后，别忘了把 [CMake][] 加入 `PATH` 中。

  [00]: http://cmake.org/cmake/resources/software.html

###使用 CMake 生成 vcproj

lua官方不提供带 CMake 的源码，会的朋友可以手工写几个 CMakeList.txt 就 OK，不会的直接下载 bitdewy 写好的 [cmake-lua-5.2.1.tar.gz][01] 吧。解压后用 visual studio 的命令行进入 `lua-5.2.1/build` 目录，然后输入：

``` bat
cmake -DCMAKE_INSTALL_PREFIX="C:\lua52" ..
```

完成之后就可以打开 build 目录下的 lua.sln 文件直接编译了，编译 INSTALL 工程，会在上面设置的 `C:\lua52` 目录中安装 lua，luac，静态链接库，动态链接库，头文件、源代码以及文档，如果不想安装，不编译 INSTALL 工程就好。如果不设置安装目录，那么 windows 下默认的安装目录就是 `%ProgramFiles%` 。

  [01]: https://y6zieq.bay.livefilestore.com/y1pXUgzupquQMAzjfw43pWLevq6HMB67BpBBAlSKajV6BdNcuOrKTpP_BgKIscW0yP7/cmake-lua-5.2.1.tar.gz
