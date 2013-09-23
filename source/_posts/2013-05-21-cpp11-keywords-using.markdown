---
layout: post
title: "C++11 关键字 using"
date: 2013-05-21 00:45
comments: true
categories: C++11 
---

*这个特性, 10年前就已经有提案了, 直到 C++11 中才正式加入标准. ⊙﹏⊙b汗*

##使用小技巧来声明函数指针

在 c++03 中, 函数指针要这么写:

```cpp
typedef void (*FunctionPtr)();
```

这个声明读起来相当的费劲, C 语言的初学者会很难理解上面的 typedef 表达的意思.

在 C++11 中, 我们可以使用更易读懂的方式来表达, 利用 using 关键字:

```cpp
using FunctionPtr = void (*)();
```

如果你想去除丑陋的 * 号, 还可以利用类型特化这么来写:

```cpp
#include <type_traits>
using FunctionPtr = std::add_pointer<void()>::type;
```

新关键字 using 当然不仅仅是为了这个简单的功能了, 最主要的目的是为了模板别名.

<!-- more -->
##模板别名

在进入这个主题之前，应该先弄清楚“模板类”和“类模板”本质上的不同。class template (类模板，是模板)是用来产生 template class (模板类，是类型)的。
在标准 C++, typedef 可定义模板类一个新的类型名称, 但是不能够使用 typedef 来定义模板的别名. 举例来说:

```cpp 
template< typename first, typename second, int third>
class SomeType;
template< typename second>
typedef SomeType<OtherType, second, 5> TypedefName; // 在C++03是不合法的
```

这不能够通过编译. 
为了定义模板的别名, C++11 增加了以下的语法:

```cpp
template< typename first, typename second, int third>
class SomeType;
template< typename second>
using TypedefName = SomeType<OtherType, second, 5>;
```

using 紧跟着的标识, 用来表示后面的模板类型. 标准制定时尝试过使用旧的关键字 typedef 来实现该功能, 但是无法得到一个完整连贯的解决方案, 因此引入了新的关键字. (严格说来 using 并不是一个新的关键字, 在使用 namespace 时, 我们就已经见过它了, 但在 C++11 中它又有了新的语义.）

 
除了模板别名之外, using 关键字还可以用在一般的类型中, 就想文章一开始的地方一样:

```cpp
typedef void (*PFD)(double);            // 传统语法
using PFD = void (*)(double);           // 新增与否
```

##参考

- C++ standard  14.6.7 Template aliases;  7.1.3 The typedef specifier
- [Templates aliases for C++](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2003/n1489.pdf)
- [Templates Aliases (Revision 3)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2258.pdf)
