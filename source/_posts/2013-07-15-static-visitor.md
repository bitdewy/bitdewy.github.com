---
layout: post
title: "Visitor 模式与 boost.variant.static_visitor"
date: 2013-07-15 00:36
comments: true
categories: [design pattern, boost]
---

##visitor 模式
 
visitor 模式是一种将算法与对象结构分离的软件设计模式.
 
这个模式的基本想法如下: 首先我们拥有一个由许多对象构成的对象结构, 这些对象的类都拥有一个 accept 方法用来接受访问者对象; 访问者是一个接口, 它拥有一个 visit 方法, 这个方法对访问到的对象结构中不同类型的元素作出不同的反应; 在对象结构的一次访问过程中, 我们遍历整个对象结构, 对每一个元素都实施 accept 方法, 在每一个元素的 accept 方法中回调访问者的 visit 方法, 从而使访问者得以处理对象结构的每一个元素. 我们可以针对对象结构设计不同的实在的访问者类来完成不同的操作.
 
访问者模式使得我们可以在传统的单分派语言 (如 Smalltalk, Java 和 C++) 中模拟双分派技术.

<!-- more -->
##传统的 visitor 模式实现

```cpp
#include <memory>
#include <iostream>
#include <tuple>
 
/// original visitor pattern
class Wheel;
class Engine;
class Body;
class Car;
 
class CarElementVisitor
{
public:
  virtual void visit(const std::shared_ptr<Wheel>& wheel) const = 0;
  virtual void visit(const std::shared_ptr<Engine>& engine) const = 0;
  virtual void visit(const std::shared_ptr<Body>& body) const = 0;
  virtual void visit(const std::shared_ptr<Car>& car) const = 0;
};
 
class CarElement
{
public:
  virtual void accept(const std::shared_ptr<CarElementVisitor>& visitor) = 0;
};
 
class Wheel : public CarElement,
              public std::enable_shared_from_this<Wheel>
{
public:
  virtual void accept(const std::shared_ptr<CarElementVisitor>& visitor)
  {
    visitor->visit(shared_from_this());
  }
};
 
class Engine : public CarElement,
               public std::enable_shared_from_this<Engine>
{
public:
  virtual void accept(const std::shared_ptr<CarElementVisitor>& visitor)
  {
    visitor->visit(shared_from_this());
  }
};
 
class Body : public CarElement,
             public std::enable_shared_from_this<Body>
{
public:
  virtual void accept(const std::shared_ptr<CarElementVisitor>& visitor)
  {
    visitor->visit(shared_from_this());
  }
};
 
class Car : public CarElement,
            public std::enable_shared_from_this<Car>
{
  typedef std::tuple<std::shared_ptr<Wheel>, std::shared_ptr<Engine>, std::shared_ptr<Body> > Elements;
public:
  Car()
  {
    elements_ = std::make_tuple(
      std::make_shared<Wheel>(), std::make_shared<Engine>(), std::make_shared<Body>());
  }
 
  virtual void accept(const std::shared_ptr<CarElementVisitor>& visitor)
  {
    visitor->visit(std::get<0>(elements_));
    visitor->visit(std::get<1>(elements_));
    visitor->visit(std::get<2>(elements_));
    visitor->visit(shared_from_this());
  }
 
private:
  Elements elements_;
};
 
class PrintVisitor : public CarElementVisitor
{
public:
  virtual void visit(const std::shared_ptr<Wheel>& wheel) const
  {
    std::cout << "void PrintVisitor::visit(std::shared_ptr<Wheel> wheel) const" << std::endl;
  }
  virtual void visit(const std::shared_ptr<Engine>& engine) const
  {
    std::cout << "void PrintVisitor::visit(std::shared_ptr<Engine> engine) const" << std::endl;
  }
  virtual void visit(const std::shared_ptr<Body>& body) const
  {
    std::cout << "void PrintVisitor::visit(std::shared_ptr<Body> body) const" << std::endl;
  }
  virtual void visit(const std::shared_ptr<Car>& car) const
  {
    std::cout << "void PrintVisitor::visit(std::shared_ptr<Car> car) const" << std::endl;
  }
};
 
class OtherVisitor : public CarElementVisitor
{
public:
  virtual void visit(const std::shared_ptr<Wheel>& wheel) const
  {
    std::cout << "void OtherVisitor::visit(std::shared_ptr<Wheel> wheel) const" << std::endl;
  }
  virtual void visit(const std::shared_ptr<Engine>& engine) const
  {
    std::cout << "void OtherVisitor::visit(std::shared_ptr<Engine> engine) const" << std::endl;
  }
  virtual void visit(const std::shared_ptr<Body>& body) const
  {
    std::cout << "void OtherVisitor::visit(std::shared_ptr<Body> body) const" << std::endl;
  }
  virtual void visit(const std::shared_ptr<Car>& car) const
  {
    std::cout << "void OtherVisitor::visit(std::shared_ptr<Car> car) const" << std::endl;
  }
};
 
int main()
{
  std::shared_ptr<CarElement> car(std::make_shared<Car>());
  car->accept(std::make_shared<PrintVisitor>());
  car->accept(std::make_shared<OtherVisitor>());
  return 0;
}
```

##boost.variant.static_visitor 的实现

```cpp
#include <iostream>
#include <boost/variant.hpp>
#include <boost/variant/static_visitor.hpp>
#include <boost/variant/apply_visitor.hpp>
 
class Wheel {};
class Engine {};
class Body {};
class Car {};
 
/// boost static visitor
 
typedef boost::variant<Wheel, Engine, Body, Car> CarElement;
 
class PrintVisitor : public boost::static_visitor<void>
{
public:
  void operator()(const Wheel& wheel) const
  {
    std::cout << "void PrintStaticVisitor::operator()(const Wheel&) const" << std::endl;
  }
 
  void operator()(const Engine& engine) const
  {
    std::cout << "void PrintStaticVisitor::operator()(const Engine&) const" << std::endl;
  }
 
  void operator()(const Body& body) const
  {
    std::cout << "void PrintStaticVisitor::operator()(const Body&) const" << std::endl;
  }
 
  void operator()(const Car& car) const
  {
    /// bla bla bla
    std::cout << "void PrintStaticVisitor::operator()(const Car&) const" << std::endl;
  }
};
 
class OtherVisitor : public boost::static_visitor<void>
{
public:
  void operator()(const Wheel& wheel) const
  {
    std::cout << "void OtherStaticVisitor::operator()(const Wheel&) const" << std::endl;
  }
 
  void operator()(const Engine& engine) const
  {
    std::cout << "void OtherStaticVisitor::operator()(const Engine&) const" << std::endl;
  }
 
  void operator()(const Body& body) const
  {
    std::cout << "void OtherStaticVisitor::operator()(const Body&) const" << std::endl;
  }
 
  void operator()(const Car& car) const
  {
    /// bla bla bla
    std::cout << "void OtherStaticVisitor::operator()(const Car&) const" << std::endl;
  }
};
 
int main()
{
  CarElement x = Car();
  PrintVisitor print_visitor;
  OtherVisitor other_visitor;
 boost::apply_visitor(print_visitor, x);
  boost::apply_visitor(other_visitor, x);
 
  return 0;
}
```

##参考

- [boost.variant.static_visitor](http://www.boost.org/doc/libs/1_54_0/doc/html/variant/reference.html#header.boost.variant.static_visitor_hpp)
- [维基百科: visitor 模式](http://en.wikipedia.org/wiki/Visitor_pattern)
