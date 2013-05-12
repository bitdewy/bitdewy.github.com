---
layout: post
title: "空基类优化"
date: 2013-02-25 7:50
comments: true
categories: [C++, idiom]
---

##目的

优化空类数据成员的存储空间


##别名

EBCO: Empty Base Class Optimization
Empty Member Optimization


##动机

大小为 0 的类在 C++ 中是不存在的. C++ 需要空类大小不为 0 以确保对象的标识. 例如下面的 `EmptyClass` 的大小就是非 0 的, 因为数组中每一个对象的标识都是唯一的. 如果 `sizeof(EmptyClass)` 的大小为 0, 指针算数就会失效. 一般情况下, 类似 `EmptyClass` 的类大小通常为 1.

```cpp
class EmptyClass {};
EmptyClass arr[10]; // Size of this array can’t be zero.
```

当类似的类作为另一个类的数据成员时, 它的大小一般比 1 字节要大. 编译器通常 4 字节对齐来避免切割. 4 字节的空类对象只是占位符, 毫无用处. 避免浪费空间, 节省内存, 帮助对象更适应 CPU 缓存是非常有好处的.

<!-- more -->
##解决方案&示例代码

在 C++ 中, 如果一个空类作为基类被继承时, 情况会和上面的有些区别. 编译器允许继承层次结构扁平化, 被继承的空基类不占用空间. 例如下面的代码中 `sizeof(AnInt)` 在 32 位架构中是 4 字节, `sizeof(AnotherEmpty)` 是 1 字节, 虽然这两个类都继承自 `EmptyClass`

```cpp
class AnInt : public EmptyClass
{
   int data;
};   // size = sizeof(int)
class AnotherEmpty : public EmptyClass {};  // size =  1
```

EBCO 有效的利用了这个特性. 这并不是说简单, 天真的将数据成员的空类变成基类是可取的, 因为这可能会暴露原本需要对用户隐藏的接口. 例如下面的 EBCO 实现方式, 可能会有副作用: 类 `Foo` 的用户现在可以看到一些方法 (如果在 E1, E2 中存在的话), 虽然他们是私有继承而来的, 不可访问.

```cpp
class E1 {};
class E2 {};
// **** before EBCO ****
class Foo {
  E1 e1;
  E2 e2;
  int data;
}; // sizeof(Foo) = 8
// **** after EBCO ****
class Foo : private E1, private E2 {
  int data;
}; // sizeof(Foo) = 4
```

一种实现 EBCO 的实用方法是: 将空的类成员组合到单一的存储结构扁平的成员中. 下面的模板 `BaseOptimization` 的前两个模板参数是用来实现 EBCO的. 使用 `BaseOptimization` 改写类 `Foo` 如下:

```cpp
template <class Base1, class Base2, class Member>
struct BaseOptimization : Base1, Base2 
{
   Member member;
   BaseOptimization() {}
   BaseOptimization(Base1 const& b1, Base2 const & b2, Member const& mem)
       : Base1(b1), Base2(b2), member(mem) { }
   Base1 * first()  { return this; }
   Base2 * second() { return this; }
};
 
class Foo {
  BaseOptimization<E1, E2, int> data;
}; // sizeof(Foo) = 4
```

使用 EBCO 并没有改变类 Foo 的继承体系. 保证基类不会互相冲突, 这是至关重要的. 也就是说 Base1 与 Base2 是独立的集成体系中的一部分.

**注意**: 对象身份标识的问题, 不同的编译器处理起来是不相同的. 空对象的地址可能是相同的, 也可能是不同的. 例如: BaseOptimization 的成员函数 first 和 second 返回的指针, 在有些编译器上可能是相同的, 而有些编译器上可能是不同的. 更多的讨论请看这里：[stackoverflow][00]

  [00]: http://stackoverflow.com/questions/7694158/boost-compressed-pair-and-addresses-of-empty-objects


##已知的用途

[boost::compressed_pair][01] 使用了该惯用法, 来优化 pair 的存储空间
C++03 模拟实现 [unique­_ptr][02] 也使用了该惯用法

  [01]: http://www.boost.org/doc/libs/1_53_0/libs/utility/compressed_pair.htm
  [02]: http://home.roadrunner.com/~hinnant/unique_ptr03.html


##参考

- [The Empty Base Class Optimization (EBCO)](http://www.informit.com/articles/article.aspx?p=31473&seqNum=2)
- [The "Empty Member" C++ Optimization](http://www.cantrip.org/emptyopt.html)
- [Internals of boost::compressed_pair](http://www.ibm.com/developerworks/aix/library/au-boostutilities/index.html)
