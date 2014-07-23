---
layout: post
title: "《Effecticve Modern C++》预览"
date: 2014-07-21 22:14
comments: true
categories: [C++]
---

虽然最近工作在用 javascript, 但 C++ 也没放下, [Scott Meyers][scott] 的新书《Effective modern C++》终于在 O'Reilly [上架了][oreilly]. 第一时间买了一本. 2年前的时候, [Scott][scott] 就给了[一版][ec11] Effective C++11 的初步想法, 两年后的今天, 顺带着 C++14 终于正式出炉了, 不过目前还是 early release 版本, 还没有经过 review, 正式版要到年底才会放出.

  [scott]: http://www.aristeia.com
  [oreilly]: http://shop.oreilly.com/product/0636920033707.do
  [ec11]: http://blog.bitdewy.me/blog/2012/06/24/effective-c-plus-plus-11/

不得不说, C++ 现在终于有了它该有的样子. 大致浏览了一下, 如果现在才开始关注 C++11/C++14 的话, 非常推荐这本书. 如果看过前两代 《Effective C++》和《More effective C++》的话, 那你会再次找到久违的感觉. 

全书共分为 6 章, 包括: 类型推导, 新关键字 auto, C++98 到 C++11/14 的变化, 智能指针, 右值引用转移语义与完美转发, lambda 表达式, 并发 API.

关于类型推导, 由于 C++11/14 引入了新的关键字 `auto`, `decltype`, 以及新的概念右值引用, 那么类型推导就不仅仅存在于模板中了, 因此从现在开始掌握类型推导的规则是 C++ 的必备技能了.

C++11/14 和以前相比, 变化非常的大, 在这本书中, 用了近 1/3 的章节来讲述如何从 C++98 过度到 C++11/14. 有些甚至推翻了之前的推荐做法, 比如 Item 13: Prefer const_iterators to iterators 就与作者之前的 《Effective STL》中的 item 26 完全相反. 所以对于每个写 C++ 代码的人来说, 这一部分是必须要更新的知识.

对于智能智能, 相信大家早就不陌生, 作者的《More Effective C++》中也讲到了智能指针. 这一次, 终于成为了标准, 本书中也讲到了各种类型智能指针合适的使用场景, 掌握了这些之后, 相信资源管理的问题, 就能够减少很多了.

转移语义很好的解决了性能问题, 完美转发解决了模板函数重载时爆炸式增长的问题, 而右值引用就是这两个特性的基础. 对于那些写库的人来说, 这些特性真是天大的好事.

关于 lambda 表达式, 有了 lambda 表达式, 标准库中的那些算法终于能变得好用了. 在没有 lambda 之前, 在使用每个标准库算法之前, 还需要一个仿函数, 这是一件多么不爽的事情.

对于并发, 在 C++98 以及之前, 标准连多线程的概念都没有, 我们只能使用平台相关的多线程设施, 甚至一些我们觉得没问题的多线程模型都会在这种情况下出问题. (比如: 这篇[C++ and the Perils of Double-Checked Locking][DDJ]) 现在标准中不仅有了 thread, 甚至还有了更高级的 task, future, promise. 当然如果能有 [Resumable Functions][n3650] 就更好了, 不过这个特性可能要等到 C++17 了.

  [DDJ]: http://www.aristeia.com/Papers/DDJ_Jul_Aug_2004_revised.pdf
  [n3650]: http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2013/n3650.pdf

<!-- more -->
最后放出《effecticve modern C++》的目录

#### Chapter 1 Deducing Types

- Item 1: Understand template type deduction.
- Item 2: Understand auto type deduction.
- Item 3: Understand decltype.
- Item 4: Know how to view deduced types.

#### Chapter 2 auto

- Item 5: Prefer auto to explicit type declarations.
- Item 6: Be aware of the typed initializer idiom.

#### Chapter 3 From C++98 to C++11 and C++14

- Item 7: Distinguish () and {} when creating objects.
- Item 8: Prefer nullptr to 0 and NULL.
- Item 9: Prefer alias declarations to typedefs.
- Item 10: Prefer scoped enums to unscoped enums.
- Item 11: Prefer deleted functions to private undefined ones.
- Item 12: Declare overriding functions override.
- Item 13: Prefer const_iterators to iterators.
- Item 14: Use constexpr whenever possible.
- Item 15: Make const member functions thread-safe.
- Item 16: Declare functions noexcept whenever possible.
- Item 17: Consider pass by value for cheap-to-move parameters that are always copied.
- Item 18: Consider emplacement instead of insertion.
- Item 19: Understand special member function generation.

#### Chapter 4 Smart Pointers

- Item 20: Use std::unique_ptr for exclusive-ownership resource management.
- Item 21: Use std::shared_ptr for shared-ownership resource management.
- Item 22: Use std::weak_ptr for std::shared_ptr-like pointers that can dangle.
- Item 23: Prefer std::make_unique and std::make_shared to direct use of new.
- Item 24: When using the Pimpl Idiom, define special member functions in the implementation file.

#### Chapter 5 Rvalue References, Move Semantics, and Perfect Forwarding

- Item 25: Understand std::move and std::forward.
- Item 26: Distinguish universal references from rvalue references.
- Item 27: Use std::move on rvalue references, std::forward on universal references.
- Item 28: Avoid overloading on universal references.
- Item 29: Familiarize yourself with alternatives to overloading on universal references.
- Item 30: Understand reference collapsing.
- Item 31: Assume that move operations are not present, not cheap, and not used.
- Item 32: Familiarize yourself with perfect forwarding failure cases.

#### Chapter 6 Lambda Expressions

- Item 33: Avoid default capture modes.
- Item 34: Use init capture to move objects into closures.
- Item 35: Use decltype on auto&& parameters to std::forward them.
- Item 36: Prefer lambdas to std::bind.

#### Chapter 7 The Concurrency API

- Item 37: Prefer task-based programming to thread-based.
- Item 38: Specify std::launch::async if asynchronicity is essential.
- Item 39: Make std::threads unjoinable on all paths.
- Item 40: Be aware of varying thread handle destructor behavior.
- Item 41: Consider void futures for one-shot event communication.
- Item 42: Use std::atomic for concurrency, volatile for special memory.

有时间的话, 后续会持续更新每一章节的具体内容.
