---
layout: post
title: "C++11 完美转发"
date: 2013-07-08 22:09
comments: true
categories: [C++, C++11]
---

##动机

在泛型编码中经常出现的一个问题是: 如何将一组参数原封不动的转发给另一个函数? 这里的原封不动指的是: 保持参数的左值 (右值), const (non-const) 属性不变.

##C++03 中参数转发存在的问题

下面来看一个例子, 对于表达式 `E(a, b, …, c)` 我们希望它与 `f(a, b, …, c)` 完全等价. 在 C++03 中这是不可能的. 下面是几种设计方案, 但所有的都会在某些条件下失效.
 
最简单的, 使用引用参数:

```cpp
template <typename A, typename B, typename C>
void f(A& a, B& b, C& c) {
    E(a, b, c);
}
```

很遗憾, 函数 f 无法处理临时变量, 例如 `f(1, 2, 3)`; 会编译失败. 这三个参数都无法绑定到引用.
 
再来看看 const 引用:

```cpp
template <typename A, typename B, typename C>
void f(const A& a, const B& b, const C& c) {
    E(a, b, c);
}
```

这解决了上面的问题, 但是又引入了新的问题, 现在我们无法让 E 接收 non-const 参数了, 下面的代码也会产生一个编译失败.

```cpp
int i = 1, j = 2, k = 3;
void E(int&, int&, int&);
f(i, j, k); // oops! E cannot modify these
```

嗯, 再来个尝试, 解决上面的问题, 我们接收 const 引用, 然后使用 const_cast 来去掉 const 属性.

```cpp
template <typename A, typename B, typename C>
void f(const A& a, const B& b, const C& c) {
    E(const_cast<A&>(a), const_cast<B&>(b), const_cast<C&>(c));
}
```

最后来终极解决方案吧, 利用重载, 根据重载决议, 我们可以使用 const 引用和 non-const 引用来进行参数重载, 对于上面的例子, 我们的实现如下:

```cpp
template <typename A, typename B, typename C>
void f(A& a, B& b, C& c);
 
template <typename A, typename B, typename C>
void f(const A& a, B& b, C& c);
 
template <typename A, typename B, typename C>
void f(A& a, const B& b, C& c);
 
template <typename A, typename B, typename C>
void f(A& a, B& b, const C& c);
 
template <typename A, typename B, typename C>
void f(const A& a, const B& b, C& c);
 
template <typename A, typename B, typename C>
void f(const A& a, B& b, const C& c);
 
template <typename A, typename B, typename C>
void f(A& a, const B& b, const C& c);
 
template <typename A, typename B, typename C>
void f(const A& a, const B& b, const C& c);
```

n 个参数的函数, 就需要 2^n 个重载版本, 这太恐怖了, 我们非常需要自动完成这些工作. 这个问题在 C++11 中的到了完美的解决.


##完美转发

C++11 新标准给了我们修复这个问题的机会 (这里有一个破坏性的解决方案, 虽然解决了完美转发的问题, 但是破坏了现有的类型推导规则[《Perfect Forwarding in C++03》](http://stackoverflow.com/questions/3591832/perfect-forwarding-in-c03/3591909#3591909)).

C++11 中引入了新的概念, 右值引用. 我们可以在不破坏现有代码的情况下, 定义右值引用的推导规则, 来解决完美转发的问题.
 
看下面的表格, (TR 表示 T 引用类型)

```cpp
TR   R
 
T&   &  -> T&  // lvalue reference to cv TR -> lvalue reference to T
T&   && -> T&  // rvalue reference to cv TR -> TR (lvalue reference to T)
T&&  &  -> T&  // lvalue reference to cv TR -> lvalue reference to T
T&&  && -> T&& // rvalue reference to cv TR -> TR (rvalue reference to T)
```

然后, 模板参数推导时, 如果参数是左值, 那么就推导为左值引用. 否则使用正常的类型推导. 这里又引入了一个新的概念, “全局引用” (universal references), 具体细节详见: [C++ and Beyond 2012: Scott Meyers - Universal References in C++11](http://channel9.msdn.com/Shows/Going+Deep/Cpp-and-Beyond-2012-Scott-Meyers-Universal-References-in-Cpp11)

为什么这样就能解决问题? 因为我们保持了参数的类型: 如果参数是一个左值, 我们得到的是左值引用的参数, 否则我们得到的是右值引用类型的参数, 看下面的代码:

```cpp
template <typename T>void deduce(T&& x); 
 
int i;
deduce(i); // deduce<int&>(int& &&) -> deduce<int&>(int&)
deduce(1); // deduce<int>(int&&)
```

最后就是要做“转发”了. 需要注意的是, 在函数内参数类型是一个左值类型.

```cpp
void foo(int&);
 
template <typename T>
void deduce(T&& x){
    foo(x); // fine, foo can refer to x
}
 
deduce(1); // okay, foo operates on x which has a value of 1
```

这还不足够. foo 需要拿到和 deduce 一样的参数类型, 解决方案如下.

```cpp
static_cast<T&&>(x);
```

上面这行代码做了什么呢? 我们在 deduce 函数中, 且我们接收了一个左值类型的参数, 这意味着 T 是 A& 类型, 经过静态类型转换 A& && 仍然是 A& 类型. 因此, 加入 x 以及是一个 A& 类型, 那么我们什么也没有做, 它仍然是一个左值引用类型.
 
当我们接收的是一个右值, T 就是 A 类型, 因此我们转换后的类型为 A&&. 转换后的类型是一个右值表达式, 这样就不会再作为左值参数了. 我们保持住了参数的类型. 将上面的组合到一起, 我们就得到了“完美转发”:

```cpp
template <typename A>
void f(A&& a) {
    E(static_cast<A&&>(a));
}
```

当 f 接收到一个左值, 那么 E 也接收一个左值, 当 f 接收的是一个右值, 那么 E 接受到的也是一个右值, 完美了.
 
当然, 上面的代码是有一点丑陋. `static_cast<T&&>` 古怪而且不容易记住. 标准库中提供了一个工具函数, 叫做 forward, 它所做的事情与类型转换完全一致.

```cpp
std::forward<A>(a);
// is the same as
static_cast<A&&>(a);
```

##参考

- [C++标准提案(N1385)：The Forwarding Problem: Arguments](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2002/n1385.htm)
- [《C++0x漫谈》系列之：右值引用(或“move语意与完美转发”)(下)](http://blog.csdn.net/pongba/article/details/1697636)
