---
layout: post
title: "why make_shared ?"
date: 2014-01-12 21:01
comments: true
categories: [C++]
---

C++11 中引入了智能指针, 同时还有一个模板函数 `std::make_shared` 可以返回一个指定类型的 `std::shared_ptr`, 那与 `std::shared_ptr` 的构造函数相比它能给我们带来什么好处呢 ?

##优点

###效率更高

`shared_ptr` 需要维护引用计数的信息,

- 强引用, 用来记录当前有多少个存活的 shared_ptrs 正持有该对象. 共享的对象会在最后一个强引用离开的时候销毁( 也可能释放).
- 弱引用, 用来记录当前有多少个正在观察该对象的 weak_ptrs. 当最后一个弱引用离开的时候, 共享的内部信息控制块会被销毁和释放 (共享的对象也会被释放, 如果还没有释放的话).

如果你通过使用原始的 new 表达式分配对象, 然后传递给 shared_ptr (也就是使用 shared_ptr 的构造函数) 的话, shared_ptr 的实现没有办法选择, 而只能单独的分配控制块:

```cpp
auto p = new widget();
shared_ptr sp1{ p }, sp2{ sp1 };
```
![](http://y6bhba.bay.livefilestore.com/y1pDu8kORu85CKSKHFVBvapxrccVoMAjc8W0d6mScWwunuhqV8n25QZKgZhM6MPtFGV1c6dPgjil7g/sharedptr1.png)

如果选择使用 `make_shared` 的话, 情况就会变成下面这样:

<!-- more -->
```cpp
auto sp1 = make_shared(), sp2{ sp1 };
```

![](http://y6yska.bay.livefilestore.com/y1p-Y0XAkcMZMrSdxkTa85nShcDkVKvqhMTFVGSDc0jURLybPXTGHPdWv9CZGgcGmvZwQQvd6Y1Qdg/sharedptr2.png)

内存分配的动作, 可以一次性完成. 这减少了内存分配的次数, 而内存分配是代价很高的操作.

关于两种方式的性能测试可以看这里 [Experimenting with C++ std::make_shared](http://tech-foo.blogspot.hk/2012/04/experimenting-with-c-stdmakeshared.html)


###异常安全

看看下面的代码:
```cpp
void F(const std::shared_ptr<Lhs>& lhs, const std::shared_ptr<Rhs>& rhs) { /* ... */ }

F(std::shared_ptr<Lhs>(new Lhs("foo")),
  std::shared_ptr<Rhs>(new Rhs("bar")));
```

C++ 是不保证参数求值顺序, 以及内部表达式的求值顺序的, 所以可能的执行顺序如下:

1. new Lhs("foo"))
2. new Rhs("bar"))
3. std::shared_ptr<Lhs>
4. std::shared_ptr<Rhs>

好了, 现在我们假设在第 2 步的时候, 抛出了一个异常 (比如 out of memory, 总之, Rhs 的构造函数异常了), 那么第一步申请的 Lhs 对象内存泄露了. 这个问题的核心在于, shared_ptr 没有立即获得裸指针.

我们可以用如下方式来修复这个问题.

```cpp
auto lhs = std::shared_ptr<Lhs>(new Lhs("foo"));
auto rhs = std::shared_ptr<Rhs>(new Rhs("bar"));
F(lhs, rhs);
```

当然, 推荐的做法是使用 `std::make_shared` 来代替:

```cpp
F(std::make_shared<Lhs>("foo"), std::make_shared<Rhs>("bar"));
```

##缺点

###构造函数是保护或私有时,无法使用 make_shared

`make_shared` 虽好, 但也存在一些问题, 比如, 当我想要创建的对象没有公有的构造函数时, `make_shared` 就无法使用了, 当然我们可以使用一些小技巧来解决这个问题, 比如这里 [How do I call ::std::make_shared on a class with only protected or private constructors?](http://stackoverflow.com/questions/8147027/how-do-i-call-stdmake-shared-on-a-class-with-only-protected-or-private-const?rq=1)

###对象的内存可能无法及时回收

`make_shared` 只分配一次内存, 这看起来很好. 减少了内存分配的开销. 问题来了, `weak_ptr` 会保持控制块(强引用, 以及弱引用的信息)的生命周期, 而因此连带着保持了对象分配的内存, 只有最后一个 `weak_ptr` 离开作用域时, 内存才会被释放. 原本强引用减为 0 时就可以释放的内存, 现在变为了强引用, 若引用都减为 0 时才能释放, 意外的延迟了内存释放的时间. 这对于内存要求高的场景来说, 是一个需要注意的问题. 关于这个问题可以看这里 [make_shared, almost a silver bullet](http://lanzkron.wordpress.com/2012/04/22/make_shared-almost-a-silver-bullet/)

##参考

- [GotW #89 Solution: Smart Pointers](http://herbsutter.com/2013/05/29/gotw-89-solution-smart-pointers/)
- [cppreference.com - std::make_shared](http://en.cppreference.com/w/cpp/memory/shared_ptr/make_shared)

