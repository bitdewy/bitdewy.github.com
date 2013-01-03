---
layout: post
title: "你确定你懂 const 和 mutable ?"
date: 2013-01-03 14:38
comments: true
categories: [C++]
---

下面是一段完整的 C++11 代码;

``` cpp
#include <utility>
#include <future>
#include <vector>
using namespace std;
vector<int> v(1000);
int main()
{
  auto x = async([]{ pair<int, vector<int>> z{1, v}; });
  auto y = async([]{ pair<int, vector<int>> z{2, v}; });
  x.wait();
  y.wait();
  return 0;
}
```

上面的程序能正确同步么? 有线程安全的问题吗?

嗯, vector 是标准库的内置容器, 如果换成我们自己写的类呢? 比如下面的代码;

``` cpp
#include <utility>
#include <future>
using namespace std;
class widget {/*...*/} w1, w2;

int main()
{
  auto x = async([]{ pair<widget, widget> z{ w1, w2 }; });
  auto y = async([]{ pair<widget, widget> z{ w1, w2 }; });
  x.wait();
  y.wait();
  return 0;
}
```

`widget` 的线程安全性如何?

<!-- more -->
上面的问题先放一放, 我们来看一个简单的问题.

在 C++98 中, `const` 表示什么含义?

嗯, `const` 表示不可更改, 不过它是 logically const, 而不是 physically 或者 bitwise 的 const.

如果你对 const 的认识仅限于此, 那么可以说你不懂 const, 在 C++11 中, 你需要重新认识 const.

看一下 C++11 标准中是怎么说的吧, C++11 标准文档, 语言核心部分 1.10 第4条和第21条:

*两个表达式, 如果其中一个正在**修改**内存, 而另一个去访问或**修改**相同的内存, 那么两个表达式会发生**冲突**.*

*如果一个程序在不同的线程中包含了两个会**冲突**的动作, 至少其中一个不是原子操作, 且不会在另一个动作发生之前发生, 那么程序运行时会产生**数据竞争**. 任何的数据竞争都会导致未定义的行为.*

再来看看关于标准库部分 17.6.5.9 的第1条和第3条:

*这一节的指定要求是防止**数据竞争**.*

*C++ 标准库函数除在当前线程外, 不能直接或间接的修改对象, 除非这个对象是通过函数的非 const 参数直接或间接的访问的, 包括 `this` 指针.*

这样一来, const 的含义就变为了 read only.

下面来看一下标准中 20.2 的代码:

``` cpp
namespace std {
namespace rel_ops {
  template <class T> bool operator!=(const T&, const T&);
  template <class T> bool operator>(const T&, const T&);
  [...]

template <class T> bool operator!=(const T& x, const T& y);
  [...] Returns: !(x == y)
```

返回值 `!(x == y)` 意味着什么?

这意味着, 使用 std::rel_ops 的任何类型都需要提供线程安全的 `operator==` 和 `operator<`. 比如, 在不同的线程中, 比较同一个对象时, 不需要串行化访问.

再来看看 20.3.2 的 `std::pair`:

``` cpp
namespace std {
  template <class T1, class T2> struct pair {
    pair(const T1& x, const T2& y);
```

构造函数的参数 `const T1& x, const T2& y` 这意味着什么?

这意味着, 任何支持拷贝, 会作为 `std::pair` 类型参数的类型 T, 都需要支持线程安全的拷贝. 比如, 多个线程同时拷贝同一个对象时, 不需要串行化访问.

上面说的这些是什么意思?

线程安全:

- bitwise const, 或者
- 类似 leaf-level locking(像 std::atomics) 的内部串行化 (非 leaf level 的 locking 可能会导致死锁)

标准库保证了 const == 线程安全:

- 所有的标准库中的类, 以及
- 所有能作用到用户自定义类型的操作符

所以就照着标准库来做吧.

任何一个标准库中定义的类, 如果声明一个该类型的 const 对象, 那么它就是线程安全的. (完全不可更改, 或者有内部的串行化访问)

在 C++98 中, const 意味着逻辑上的不可更改. 而在 C++11 中, const 意味着线程安全(内存完全不可改, 或者内部实现了串行化访问).

明白了 const, 接下来看看下面的代码有什么问题:

``` cpp
class widget {
  mutex m;  // protects internal data
  //...     // more data

public:
  info get_info() const {
    lock_guard<mutex> hold(m);
    return {/* use 'more data'*/};
  }
};
```

是的, 上面的代码编译不过. `lock_guard` 需要的是 `mutex&` 类型, 而现在的类型是 `const mutex&`.

那么该如何修改呢? 像下面这样么?

``` cpp
class widget {
  mutex m;  // protects internal data
  //...     // more data

public:
  info get_info() const {
    lock_guard<mutex> hold(const_cast<mutex&>(m));
    return {/* use 'more data'*/};
  }
};
```

这么改, 没有任何问题, 程序可以正确的运行, 但是, 我们需要在使用了 m 的地方都用一下 `const_cast`. 包括以后新增的代码也需要, 这不是个好办法.

改成下面这样呢?

``` cpp
class widget {
  mutable mutex m;  // protects internal data
  //...             // more data

public:
  info get_info() const {
    lock_guard<mutex> hold(m);
    return {/* use 'more data'*/};
  }
};
```

回顾一下, `mutable` 在 C++98 中的含义: "mutable 意味着不会被观察到的非 const."

如果你对 mutable 的认识仅限于此, 那么可以说你不懂 mutable, 在 C++11 中, 你需要重新认识 mutable.

一个类中的 mutex 对象想要成为 mutable. 为什么?

**因为 mutex 自己已经是串行化访问的了**.

在 C++98 中, mutable 意味着不会被观察到的非 const, 而在 C++11 中, mutable 意味着线程安全.

现在在来看上面的代码, mutable表达了什么:

线程安全, 而且是自然而且完美的表达, 因为 mutable 通常就是意味着 "as good as const/ logically const".

再来看一个例子:

``` cpp
class widget {
  atomic<int> counter_;            // protects internal data
  //...                            // more data

public:
  info get_info() const {
    ++counter_;                    // oops, const
    return {/* use 'more data'*/};
  }
};
```

与最开始 mutex 的例子一样, 这段代码编译不过. 我们要做的就是把 `counter_` 声明为 `mutable`.


###参考

- [C++ and Beyond 2012: Herb Sutter - You don't know [blank] and [blank]](http://channel9.msdn.com/posts/C-and-Beyond-2012-Herb-Sutter-You-dont-know-blank-and-blank)
