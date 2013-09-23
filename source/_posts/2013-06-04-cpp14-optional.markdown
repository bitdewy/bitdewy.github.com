---
layout: post
title: "C++14 std::optional<T>"
date: 2013-06-04 01:54
comments: true
categories: [C++, C++14]
---

C++11已经发布将近2年时间, 各编译器也陆续有了较好的支持. C++14 也已经有相当多的提案被接受, 包括 std::optional, 泛型 lambda, make_unique, 动态数组等, 肯定会出现在下一个标准中. 关于新标准中的提案, 以及已经被接受的提案，详细信息可以参考, wikipedia：[C++14][wiki], 以及 open-std 中的 [C++ 标准草案][draft].

  [wiki]: http://en.wikipedia.org/wiki/C%2B%2B14
  [draft]: http://www.open-std.org/jtc1/sc22/wg21/docs/papers/
  
##“无意义”的值

函数并不总能返回有效的返回值, 很多时候函数可能返回"无意义"的值, 这不意味着函数执行失败, 而是表明函数正确执行了, 但结果却不是有用的值.
表示返回值无意义最常用的做法是增加一个"哨兵"的角色, 它位于解空间之外, 比如 `NULL, -1, EOF, std::string::npos, vector::end()`等. 但这些做法不够通用, 而且很多时候不存在解空间之外的"哨兵".
 
##std::optional<T>

optional 库使用"容器"语义, 包装了"可能产生无效值"的对象, 实现了"未初始化"的概念. (在新标准未正式发布之前，可以参考 [boost::optional][boost.optional] 的实现.)
 
optional 使用"容器"语义, 为这种"无效值"的情形提供了一个较好的解决方案. 它很像一个仅能存放一个元素的容器, 它实现了"未初始化"的概念: 如果元素未初始化, 那么容器就是空的, 否则, 容器内就是有效的, 已经初始化的值.

  [boost.optional]: http://www.boost.org/doc/libs/1_54_0/libs/optional/doc/html/index.html

<!-- more  -->
##操作函数

optional 的模板类型参数 T 可以使任何类型， 就如同一个标准容器对元素的要求, 并不需要T具有缺省构造函数, 但必须是可拷贝构造的.
optional 采用了指针语义来访问内部保存的元素, 这使得 optional 未初始化时的行为就像一个空指针. 它重载了 `operator*` 和 `operator->` 以实现与指针相同的操作, `get()` 和 `get_ptr()` 可以以函数的操作形式获得元素的引用和指针. 成员函数 `get_value_or(default)` 是一个特别的访问函数, 可以保证返回一个有效的值, 如果 optional 已初始化, 那么返回内部的元素, 否则返回 default. optional 也可以用隐式类型转换进行 bool 测试(用于条件判断), 就像一个对指针的判断. optional 还全面支持比较运算，包括 `==, !=, <, <=, >, >=`. (详细接口参考 [cppreference: std::optional][std.optional])
 
##用法

optional 的接口简单明了, 把它认为是一个大小为 1 并且行为类似指针的容器就可以了, 或者把它想象成是一个类似 scoped_ptr, shared_ptr 的智能指针(注意, optional 不是智能指针, 用法类似但用途不同).

```cpp 
#include <iostream>
#include <string>
#include "boost/optional.hpp"
#include "boost/lexical_cast.hpp"
 
// converts int to string if possible
boost::optional<int> str2int(const std::string& str)
{
  boost::optional<int> i;
  try {
    i = boost::lexical_cast<int>(str);
  } catch (boost::bad_lexical_cast&) {
 
  }
  return i;
}
 
int get_int_form_user()
{
  std::string s;
  for (;;) {
    std::cin >> s;
    boost::optional<int> o = str2int(s); // 'o' may or may not contain an int
    if (o) {                                 // does optional contain a value?
      return *o;                             // use the value
    }
  }
}
 
int main()
{
  std::cout << get_int_form_user();
  return 0;
}
```

如上面的代码所示: 使用 optional, 非法的 int 类型, 就不需要定义一个特别的错误码作为无意义值了.
 
##参考
- [cppreference.com: optional][std.optional]
- [A proposal to add a utility class to represent optional objects (Revision 4)][draft]

  [std.optional]: http://en.cppreference.com/w/cpp/utility/optional
  [draft]: http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2013/n3672.html

