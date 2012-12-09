---
layout: post
title: "COM、ABI与引用计数"
date: 2012-12-09 18:18
comments: true
categories: [C++, COM]
---

##什么是 COM ？

Component Object Model (COM) 组件对象模型，是微软 1993 年引入的软件组件的二进制接口标准。它可以让多种编程语言之间可以相互通信，动态的创建对象。详细内容请看 [Component Object Model][wikiCOM]。

  [wikiCOM]: http://en.wikipedia.org/wiki/Component_Object_Model


##什么是 ABI ？

Application Binary Interface (ABI) 应用二进制接口，描述了应用程序和操作系统之间，一个应用和它的库之间，或者应用的组成部分之间的低层接口。

ABI涵盖了各种细节，例如：

- 数据类型、大小以及内存布局
- 调用约定（控制着函数的参数如何传送以及如何接受返回值）
- 系统调用的编码和一个应用如何向操作系统进行系统调用
- 在一个完整的操作系统ABI中，目标文件、程序库的二进制格式等等。

一个完整的ABI，像Intel二进制兼容标准([iBCS][])，允许支持它的操作系统上的程序不经修改在其他支持此ABI的操作体统上运行。

其他的ABI标准化细节包括[C++ name mangling][00]、异常传播，同一个平台上的编译器之间的调用约定，但是不包括跨平台的兼容性。详细内容请看 [Application binary interface][wikiABI]

  [00]: http://en.wikipedia.org/wiki/Name_mangling#Name_mangling_in_C.2B.2B
  [iBCS]: http://www.everything2.com/index.pl?node=iBCS
  [wikiABI]: http://en.wikipedia.org/wiki/Application_binary_interface


<!-- more -->
##什么是引用计数？

引用计数是一种资源管理的方式，经常和垃圾回收在一起讨论，引用计数策略和[垃圾回收][GC]策略都属于资源的自动化管理 ，在引用计数中，每一个对象负责维护对象所有引用的计数值。当一个新的引用指向对象时，引用计数器就递增，当去掉一个引用时，引用计数就递减。当引用计数到零时，该对象就将释放占有的资源。COM 是使用引用计数的典型例子之一。详细内容请看 [Reference counting][RC#COM]

  [GC]: http://en.wikipedia.org/wiki/Garbage_collection_(computer_science)
  [RC#COM]: http://en.wikipedia.org/wiki/Reference_counting#COM


##为毛 COM 中的接口都没有虚析构函数？

这是个违反直觉的设计：

1. 《Effective C++》 第三版，item 7：为多态基类声明 virtual 析构函数
2. 《C++ Coding Standards -- 101 Rules, Guidelines, and Best Practices》第 50 条，将基类析构函数设为公用且虚拟的，或者保护且非虚拟的

学过 C++ 的人应该都知道，基类需要声明为 virtual 或者禁止直接调用基类的析构函数，否则使用派生类初始化基类指针，当调用 delete 释放时，由于基类的析构函数不是虚函数，那么派生类的析构函数将不会被调用，造成对象的切割，派生类特有的部分将不会得到释放，造成内存或资源的泄漏。

COM 的核心在于：接口，它解决了二进制级复用的两个主要问题：

1. 不同的编译器对具体技术的不同实现问题和[name mangling][10]问题。首先，客户程序源代码中仅仅需要引入接口定义，而不同的编译器对同一接口的VTBL的结构安排是一样的，所有的组件功能的调用都通过同样的VTBL来中转。其次，由于用户通过接口来调用组件的功能，而不需要其它导出函数，所以没有[name mangling][10]的问题了。
2. 组件仅仅导出接口，而不是导出类，避免了因组件中的类的大小发生变化(破坏了二进制兼容性)而客户程序不重新编译而继续运行时产生运行错误的问题。

  [10]: http://en.wikipedia.org/wiki/Name_mangling

不同的编译器对一个接口的vptr和vtbl是一致的，但纯虚析构函数不满足，因为不同的编译器对纯虚析构函数指针在vtbl中安放的位置是不一样的，因此 COM 要实现二进制级的复用，就不能有虚析构函数。COM 使用这种方式实现了 ABI，满足了二进制兼容的问题。

但为了实现二进制兼容，COM 没有办法原地更新而不影响现有代码，只能每次发布新版本都引入新的 interface class，然后就有了一堆带版本号的 interface：

- IDocHostUIHandler，IDocHostUIHandler2
- IDirect3D7, IDirect3D8, IDirect3D9
- IXMLDOMDocument, IXMLDOMDocument2, IXMLDOMDocument3

对于追求代码好看的人来说，这很难接受，**实在是太难看了**。不过这的确解决了二进制兼容的问题。


##为毛 COM 需要使用引用计数？

首先 windows 跨 dll 释放内存会存在严重问题，因此资源的申请以及释放都要在组建内部完成。 其次，COM 组件的生命周期不能由客户来管理，因为用户可以得到指向同一个实体的多个接口型的指针，这样对多个指针执行 delete 操作，将会导致运行时错误，并且用户必须记住哪个指针对应哪个对象，并保证对每个对象仅仅调用一次 delete 操作。为了解决这个问题，COM 将这种操作从用户移到组件内部，使用了引用计数机制，同时保证了二进制兼容性。

COM 使用引用计数的最主要动机是在不同的语言和运行时系统中都能正常使用，用户只使用相应的接口(addRef, Release)来管理对象的生命周期，而不必知道 COM 对象的内存分配细节到底如何实现。

为了实现跨语言，COM 将大量的工作都放到了组件内部来做，例如指针在继承链中转换时，会使用 RTTI，但是 RTTI 是与编译器相关的，所以这种转换的动作只能放到 COM 内部，导出一个 QueryInterface 函数来执行这种转换并返回适当的指针。

但 COM 的引用计数代价不小，使用 COM 时最容易出现的 bug 就是引用计数不正确。而引用计数的不正确有可能是在某个不透明的第三方组件中。因此， 保证引用计数的正确性不是个容易的问题。

在 .Net 中，微软抛弃了[引用计数][RC]，而引入了[垃圾回收][GC]。

  [RC]: http://en.wikipedia.org/wiki/Reference_counting

####参考

- 陈硕：[C++ 工程实践(4)：二进制兼容性](http://www.cnblogs.com/Solstice/archive/2011/03/09/1978024.html)
- wikipedia: [Component Object Model][wikiCOM]
- wikipedia: [Application binary interface][wikiABI]
