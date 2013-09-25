---
layout: post
title: "在 C++03 中实现强类型安全的枚举类型"
date: 2013-02-05 21:53
comments: true
categories: [C++, type safe]
---

##意图

改进 C++ 枚举的类型安全性


##动机

在 C++03 中枚举类型的类型安全性不够强, 有可能导致意想不到的错误. 虽然枚举是语言内置的特性, 但也可能由于编译器的不同, 而存在可移植性的问题. 枚举的问题基本上可以分为3类:

- 隐式转换
- 无法指定类型
- 作用域问题

C++03 中枚举类型是部分类型安全的, 比如你将一个枚举类型的值直接赋值给另一个枚举类型, 而且无法从整形隐式转换到枚举类型. 但是, 最常见的枚举类型错误是: 它可以自动提升成整形. 例如下面的代码, 只有少数几个的编译器（比如 GNU 的 g++）会发出警告, 而 VC++ 不会有任何的提示.

```cpp
enum color { red, green, blue };
enum shape { circle, square, triangle };
color c = red;
bool flag = (c >= triangle); // Unintended!
```

<!-- more -->
C++03 中枚举类型另外的问题是无法指定枚举存储时的类型. 这可能导致数据大小以及符号问题在各个编译器上表现不同而导致不可移植的问题. 最后, 是作用域问题, 在相同的作用域内不能有两个相同名称的枚举值.

强类型的枚举已经进入 C++11 标准中. 详细内容参考: [C++11 FAQ: enum class][cpp11_enum_class]
下面是 C++03 强类型安全的枚举实现。

  [cpp11_enum_class]: http://www.stroustrup.com/C++11FAQ.html#enum


##解决方案&示例代码

```cpp
template<typename def, typename inner = typename def::type>
class safe_enum : public def
{
  typedef typename def::type type;
  inner val;
 
public:
 
  safe_enum(type v) : val(v) {}
  inner underlying() const { return val; }
 
  bool operator == (const safe_enum & s) const { return this->val == s.val; }
  bool operator != (const safe_enum & s) const { return this->val != s.val; }
  bool operator <  (const safe_enum & s) const { return this->val <  s.val; }
  bool operator <= (const safe_enum & s) const { return this->val <= s.val; }
  bool operator >  (const safe_enum & s) const { return this->val >  s.val; }
  bool operator >= (const safe_enum & s) const { return this->val >= s.val; }
};
 
struct color_def {
  enum type { red, green, blue };
};
typedef safe_enum<color_def> color;
 
struct shape_def {
  enum type { circle, square, triangle };
};
typedef safe_enum<shape_def, unsigned char> shape; // unsigned char representation
 
int main(void)
{
  color c = color::red;
  bool flag = (c >= shape::triangle); // Compiler error.
}
```

在上述解决方案的代码中, 实际的枚举类型包裹在 `color_def` 和 `shape_def` 结构体中. `safe_enum` 模板帮我们实现了一个类型安全的枚举. `safe_enum` 使用了 [Parameterized Base Class][PBC] 惯用法. 它继承了模板参数 `color_def` 以及 `shape_def`, 使得 color 以及 shape 变成了强类型安全的枚举.

  [PBC]: http://en.wikibooks.org/wiki/More_C%2B%2B_Idioms/Parameterized_Base_Class


##参考

- [Strongly Typed Enums (revision 3)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2347.pdf)
- [Effective C++11: Content and Status -Prefer enum classes to enums.](http://scottmeyers.blogspot.hk/2013/01/effective-c11-content-and-status.html)
- [C++11 FAQ: enum class -- scoped and strongly typed enums](http://www.stroustrup.com/C++11FAQ.html#enum)

