---
layout: post
title: "Effective C++11"
date: 2012-06-24 10:50
comments: true
categories: [C++] 
---
C++应该算是流行的编程语言中最复杂的（之一）。看看这篇 [C++, Ruby, CoffeeScript: a visual comparison of language complexity][00] 就知道 C++ 相对于 Ruby 和 CoffeeScript 有多么复杂了。关于 C++ 还有段子：When your hammer is C++, everything begins to look like a thumb.（[Steve Haflich][01] in [comp.lang.lisp,January 1995][02]）。

  [00]: http://www.cpprocks.com/cpp-ruby-coffeescript-language-complexity/
  [01]: http://www.franz.com/~smh/index.html
  [02]: https://groups.google.com/forum/?fromgroups=#!msg/comp.lang.lisp/jJ9gOviA6e8/6zIV-KWjuVsJ

也许正是因为C++ 如此复杂，[Scott Meyers][meyers] 的《Effective C++》才会受到如此的推崇，几乎在所有 C++ 书籍的推荐名单上，这部专著都会位于前三名。甚至有人说 C++ 程序员可以分成两类，读过 Effective C++ 的和没读过的。可见 Effective C++ 的地位。

<!-- more -->
去年 10 月 ISO/IEC [发布了][20] C++11 语言标准，[open standards][21] 上可以找到标准草案 [n3242][22] (PDF)，（[wiki][23] 上显示最终标准是3290，不过鉴于标准文档动辄几百刀，况且几乎没有人会对着标准文档学习，大家还是看看草案就好了，相信差别不大），C++ 11 更像是一个全新的语言，学习 C++ 11 可以参考 [Bjarne Stroustrup][24] 的 [C++0xFAQ][25] ，不过还是很期待 Effective 系列，不知道大师 [Scott Meyers][meyers] 什么时候会再出 Effective C++ 11，4月份的时候 [Scott Meyers][meyers] 在 [C++ and Beyond][26] 上发出了 Effective C++ 11 的初步想法（原文在[这里][27]），先睹为快吧：

* Prefer auto to Explicit Type Declarations
* Distinguish () and {} When Creating Objects
* Remember that auto + { expr } == std::initializer_list
* Prefer non-member begin/end to member versions
* Declare std::thread Members Last in Classes
* Be Wary of Default Capture Modes in Lambdas Escaping Member Functions
* Prefer Emplacement to Insertion
* Pass std::launch::async if Asynchronicity is Essential
* Minimize use of Weak Atomics
* Distinguish Rvalue References from Universal References
* Assume that move operations are neither present nor cheap
* Prefer Lambdas over Binders
* Prefer Lambdas over Variadic Arguments to Threading Functions
* Be Wary of Oversubscription
* Apply std::forward when Passing Universal References
* Prefer std::array to Built-in Arrays
* Use std::make_shared Whenever Possible
* Prefer Pass-by-Reference-to-const to Pass-by-Value for std::shared_ptrs
* Pass by Value if You’ll Copy Your Parameter
* Reserve noexcept for Functions with Wide Interfaces
* For Copyable Types, View Move as an Optimization of Copy
* Prefer enum classes to enums
* Prefer nullptr to NULL and 0
* Distinguish among std::enable_if, static_assert, and =delete

  [20]: http://www.iso.org/iso/pressrelease.htm?refid=Ref1472
  [21]: http://open-std.org/
  [22]: http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2011/n3242.pdf
  [23]: http://en.wikipedia.org/wiki/C%2B%2B11#C.2B.2B_Standards_Committee_papers
  [24]: http://www.research.att.com/~bs
  [25]: http://www2.research.att.com/~bs/C++0xFAQ.html
  [26]: http://cppandbeyond.com/
  [27]: http://cppandbeyond.com/2012/04/16/session-topic-initial-thoughts-on-effective-c11/
  [meyers]: http://www.aristeia.com/

