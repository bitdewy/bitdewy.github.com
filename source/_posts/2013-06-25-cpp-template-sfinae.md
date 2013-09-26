---
layout: post
title: "C++ 泛型编程之 SFINAE"
date: 2013-06-25 23:01
comments: true
categories: [C++, template]
---

##什么是 SFINAE?

(Substitution Failure Is Not An Error) 匹配失败不是错误. 它可以从一组重载函数中剪裁掉不需要的模板实例.
 
SFINAE 是 C++ 模板的一个特性, 这个特性在 std::enable_if 中使用的非常广泛. 模板参数推导过程中, C++ 编译器会试图实例化一些候选的函数签名, 以确保函数调用时模板的精确匹配, 当找到一个不合适的匹配时(例如无效的参数或范围值), 则会将这个不合适的匹配从重载决议中删除, 而不是产生一个编译错误. 如果有且只有一个函数的话, 才会产生一个编译错误.

考虑下面的简单的乘法计数器

```cpp
long multiply(int i, int j) { return i * j; }
 
template <class T>
typename T::multiplication_result multiply(T t1, T t2)
{
  return t1 * t2;
}
 
int main(void)
{
  multiply(4,5);
  return 0;
}
```

在 main 函数中的函数调用 multiply, 会导致编译器对模板参数的实例化, 虽然第一个非模板函数 multiply 明显是一个更佳的匹配. 在实例化的过程中, 会产生一个非法的类型 int::multiplication_result. 根据 SFINAE, 这个非法的实例化会被自动丢弃. 最终, 非模板的 multiply 会被调用. 编译通过.

<!-- more -->
##SFINAE 的应用

SFINAE 通常被用作编译时期的类型属性检查. 比如下面的 is_pointer 元函数, 它可以在编译期检查参数类型是否是一个指针.

```cpp
template <typename T>
struct is_pointer
{
  template <typename U>
  static char is_ptr(U *);
 
  template <typename X, typename Y>
  static char is_ptr(Y X::*);
 
  template <class U>
  static char is_ptr(U (*)());
 
  static double is_ptr(...);
 
  static T t;
  enum { value = sizeof(is_ptr(t)) == sizeof(char) };
};
 
struct Foo {
  int bar;
};
 
int main(void)
{
  typedef int * IntPtr;
  typedef int Foo::* FooMemberPtr;
  typedef int (*FuncPtr)();
 
  printf("%d\n",is_pointer<IntPtr>::value);         // prints 1
  printf("%d\n",is_pointer<FooMemberPtr>::value);  // prints 1
  printf("%d\n",is_pointer<FuncPtr>::value);        // prints 1
  return 0;
}
```

如果没有 SFINAE 的话, 元函数 is_pointer 是无法正常工作的. 在 is_pointer 中, 有四个模板函数 is_ptr, 其中三个都会返回一个 char, 这是经过精心设计的. 这三个分别接收不同的参数类型, 一个接受指针变量, 一个接受一个成员指针, 以及一个简单的函数指针类型. 最后一个 is_ptr 是接收任意类型的(使用了省略号), 它的返回值类型是 double, 它的大小始终会大于 char 类型.
 
当传递一个指针类型(例如上面的 IntPtr)给 is_pointer 时, value 的值将会初始化为 true (因为 sizeof 两边是相等的). 第一个 sizeof 表达式调用 is_ptr, 如果它是一个指针类型, 那么只有一个模板的重载版本会匹配. 根据 SFINAE, 编译不会报错，因为至少有一个函数被匹配上了.  如果没有合适的实例化版本, 那么会选择那个省略号的版本. 不过那个版本会返回 double 类型, 会导致 value 值被初始化为 false (因为 sizeof(double) != sizeof(char)).
 
需要注意的是, is_ptr 只有声明, 没有定义. 因为声明足以引发 SFINAE 规则, 但是这些必须都是模板函数。一个模板类的非模板函数是不会参与 SFINAE 的, 只有模板函数才会遵循 SFINAE 规则.
 
在标准库中, 有大量的 SFINAE 应用, 比如 type traits. 有了 SFINAE 我们就可以根据自己的需求, 为我们所关心的特定类型, 写相应的函数重载函数版本, 也可以在编译期判断类型是否符合我们的需求.

##参考

- Wikipedia：[Substitution failure is not an error](http://en.wikipedia.org/wiki/Substitution_failure_is_not_an_error)

