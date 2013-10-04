---
layout: post
title: "Attorney Client"
date: 2013-08-14 00:45
comments: true
categories: [idiom]
---

##目的

控制访问一个类内部实现细节的粒度.

##动机

C++ 的友元申明可以完全访问一个类的内部细节. 所以, 唔, 友元太邪恶了, 它破坏了精心的封装. C++ 中不提供选择性的授权访问类私有成员子集的方式. 在 C++ 中要么使用友元, 可以访问一个类的全部细节, 要么不用友元, 只能访问公有接口.
 
看下面的例子, 下面的类 Foo 中声明了 Bar 友元. 因此 Bar 类可以访问 Foo 的全部私有成员. 这不可取, 因为它增加了耦合. Bar 类会变得没有 Foo 类就无法单独使用.

```cpp
class Foo
{
private:
  void A(int a);
  void B(float b);
  void C(double c);
  friend class Bar;
};

class Bar
{
// This class needs access to Foo::A and Foo::B only.
// C++ friendship rules, however, give access to all the private members of Foo.
};
```

提供选择性访问成员子集的功能, 是可取的. 这样的话, 剩余的私有成员如果需要的话, 可以更改接口. 它有助于减少两个类之间的耦合. Attorney-Client 可以精确控制它的友元能够访问的成员的数量.

<!-- more -->
##解决方案 & 示例代码
 
Attorney-Client 通过增加一个间接层来解决上面描述的问题. 想要访问一个类的实现细节, 我们需要一个 Attorney 类, 一个真正的 C++ 友元. Attorney 类是精心设计的一个代理类. 和一般的代理类不同, Attorney 类只负责复制 Foo 类的内部实现的接口的子集. 为了更好的描述, 我们把 Foo 类重新命名为 Client, Client 仅仅想通过 Attorney 提供 Client::A 以及 Client::B 的访问权限.

```cpp
class Client
{
private:
  void A(int a);
  void B(float b);
  void C(double c);
  friend class Attorney;
};

class Attorney
{
private:
  static void callA(Client & c, int a) {
    c.A(a);
  }
  static void callB(Client & c, float b) {
    c.B(b);
  }
  friend class Bar;
};

class Bar
{
// Bar now has access to only Client::A and Client::B through the Attorney.
};
```

Attorney 类限制仅能访问部分函数, Attorney 类中仅函数静态内联成员函数, 每个都需要一个 Client 类实例的引用, 然后将调用转到这个实例上. 他的实现是完全私有的. 这可以禁止其他类去访问 Client 内部. Attorney 可以决定哪些类, 成员函数, 以及自由函数可以访问 Client. 它只需要将能访问这些细节的东西声明为友元即可. 如果没有 Attorney 类, Client 也需要子集来做.

它也可以有多个 attorney 类来提供访问 Client 的不同细节. 比如 AttorneyC 类只提供 Client::C 的访问.

在下面的例子中, Attorney-client 用在了 类 Base 和 main 函数中间. Derived::Func 函数通过多态性得以访问. 不过如果想要再访问 Derived 类的一些细节的时候, 那就需要再次使用 Attorney 了.

```cpp
#include <cstdio>
class Base
{
private:
  virtual void Func(int x) = 0;
  friend class Attorney;
public:
  virtual ~Base() {}
};

class Derived : public Base
{
private:
  virtual void Func(int x)  {
    printf("Derived::Func\n"); // This is called even though main is not a friend of Derived.
  }
public:
  ~Derived() {}
};

class Attorney
{
private:
  static void callFunc(Base & b, int x) {
    return b.Func(x);
  }
  friend int main (void);
};

int main(void)
{
  Derived d;
  Attorney::callFunc(d, 10);
}
```

##已知的应用

- [Boost.Iterators library](http://www.boost.org/doc/libs/1_50_0/libs/iterator/doc/iterator_facade.html#iterator-core-access)
- [Boost.Serialization: class boost::serialization::access](http://www.boost.org/doc/libs/1_50_0/libs/serialization/doc/serialization.html#member)
 
##参考
- [Friendship and the Attorney-Client Idiom (Dr. Dobb's Journal)](http://drdobbs.com/184402053)
