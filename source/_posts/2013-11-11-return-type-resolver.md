---
layout: post
title: "返回值类型推导"
date: 2013-11-11 10:21
comments: true
categories: [C++]
---

模版函数的参数类型可以自动推导, 它可以让我们在调用函数的时候不必写丑陋的`<>`, 但如果是返回值类型需要自动推导, 似乎就没有那么容易了. 虽然语言本身不支持返回值的类型自动推导, 但我们可以尝试其他的办法来完成这项任务.

##目的

在使用函数返回值初始化变量或给变量赋值时模版函数能够自动推导出类型.


##例子

在某些情况下, 明确被初始化的变量类型是有用的. 考虑下面的例子, 我们使用随机数来初始化 STL 容器. 但是我们不知道用户会选择哪个具体的容器类型. 它可能是 `std::list`, `std::vector` 或者其他的行为像 STL 容器的自定义类型. 最直接的方法如下所示:

```cpp
#include <list>
#include <vector>
#include <random>

template <class Container>
Container GetRandomN(size_t n) {
  Container c;
  for(size_t i = 0; i < n; ++i) {
    c.insert(c.end(), rand());
  }
  return c;
}

int main () {
  std::list<int> l = GetRandomN<std::list<int> >(10);
  std::vector<long> v = GetRandomN<std::vector<long> >(100);
  return 0;
}
```
<!-- more -->
我们可以注意到, 代码中必须显示的指定返回值的类型, 很显然我们必须写两次返回值类型, 类型声明写一次, 函数调用又写了一次. 返回值类型推导可以帮助我们解决这个问题, 当然 C++11 中的 auto 类型也可以解决.

##解决方案 & 示例代码

```cpp
#include <list>
#include <set>
#include <vector>
#include <random>

class GetRandomN {
 public:
  GetRandomN(int n = 1) : count(n) {}
  
  template <class Container>
  operator Container() {
    Container c;
    for (size_t i = 0; i < count; ++i) {
      c.insert(c.end(), rand()); // push_back is not supported by all standard containers.
    }
    return c;
  }
 private:
  size_t count;
};

int main() {
  std::set<int> random_s = GetRandomN(10);
  std::vector<int> random_v = GetRandomN(10);
  std::list<int> random_l = GetRandomN(10);
  return 0;
}
```

类 GetRandomN 有一个构造函数, 以及一个模版的类型隐式转换函数. 初始化的时候, 会产生一个 GetRandomN 类的临时对象, 当赋值给 STL 容器类型时, 编译器会去尝试将对象隐式转换. 而隐式转换的途径就是取寻找隐式转换函数, 这样就完成了返回值自动推导. 有了返回值类型自动推导, 我们就不必手写返回值类型参数了. 唔… 再注意一点, 为了支持 `std::set` 我们将原始的 `push_back` 函数改为了 `insert`.

*注: 在 C++11 引入 `nullptr` 之前, 手工实现一个 `nullptr` 的惯用法就使用了返回值类型推导.*
