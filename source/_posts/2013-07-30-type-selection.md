---
layout: post
title: "类型选择"
date: 2013-07-30 18:26
comments: true
categories: [C++, metaprogramming]
---

##目的

在编译期间, 根据一些条件, 决定类型.

##动机

在编译期根据一些信息来做出决议是元编程的一个强大工具. 其中一种就是在编译期决定类型, 比如根据一些列的条件最后决定具体的类型.
 
例如, 考虑一个 Queue 的实现, 这个模板类有一个静态数组, 模板参数作为这个 Queue 类型的最大容量. Queue 类同时需要存储当前元素的个数, 从 0 开始. 可能的优化方案是用不同的类型, 来存储 size 值. 比如当 Queue 的最大容量小于 256 时, 存储容量的类型是 unsigned char, 当容量小于 65536 时, 存储容量的类型是 unsigned short, 更大的容量时使用 unsigned int 类型. 类型选择可以在这个时候发挥作用.

##解决方案以及代码示例

一个简单的解决方案是使用 IF 模板. IF 模板有三个模板参数, 第一个参数是编译期的 bool 表达式. 如果表达式为 true 那么第二个模板参数类型将被选中, 否则会选中第三个模板参数. IF 模板由下面的一个模板和一个模板偏特化构成.

```cpp
template <bool, class L, class R>
struct IF  // primary template
{
  typedef R type;
};
template <class L, class R>
struct IF<true, L, R> // partial specialization
{
  typedef L type;
};
IF<false, int, long>::type i; // is equivalent to long i;
IF<true,  int, long>::type i; // is equivalent to int i;
```
<!-- more -->
下面就是使用类型选择实现的 Queue 类:

```cpp
template <class T, unsigned int CAPACITY>
class Queue
{
  T array[CAPACITY];
  typename IF<(CAPACITY <= 256),
      unsigned char,
      typename IF<(CAPACITY <= 65536),
                  unsigned short,
                  unsigned int
                 >::type
    >::type size;
  // ...
};
```

模板类 Queue 声明了一个类型为 T 的数组, 数据成员 size 的类型由两个 IF 模板的结果来决定. 注意, 这些比较不是在运行时而是在编译期.

##已知的应用

- [Boost.MPL Library](http://www.boost.org/doc/libs/1_54_0/libs/mpl/doc/index.html)

##参考

- 《Modern C++ design》(2.6 型别选择 (type selection)) Andrei Alexandrescu

