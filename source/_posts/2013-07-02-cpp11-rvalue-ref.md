---
layout: post
title: "C++11 右值引用"
date: 2013-07-02 22:49
comments: true
categories: [C++, C++11]
---

##什么是右值引用?

在一般情况下, C++ 不把表达式的左值属性作为类型的一部分, 比如下面的变量声明:

```cpp
int n;
```

那么变量 n 的类型是 int. 另外常量 3 也是 int 类型. 虽然他们有相同的类型, 不过这并不是说 n 和 3 总是可以互换的. 比如下面的代码:

```cpp
int& r = n;  // OK
int& s = 3;  // Error
```

这是因为标准中规定, 当定义一个引用时, 我们必须使用一个左值来初始化它. 在上面的代码中, n 是一个左值, 而 3 不是. 所以我们不能用 3 来初始化 s. 这个规定可以帮助编译器来捕捉错误, 比如下面的代码:

```cpp
std::cin >> 3;  // equals std::cin.operator>>(3);
```

简单的说, 上面的代码调用了 std::cin 的名为 operator>> 的成员函数, 并使用 3 作为参数. operator>> 是重载函数, 虽然它有接受一个 int& 参数的重载版本, 但是没有能接受一个简单的 int 常量, 或者 int 右值作为参数的重载. 因此, 编译器会检查出我们无法使用 int 的右值作为参数来调用 operator>>.

<!-- more -->
C++11 扩展了引用的概念, 增加了右值引用. 右值引用也是一个引用, 但与普通的引用不同, 它只能绑定到右值上. 下面是一个右值引用的声明:

```cpp
int&& t = 3;
```

一个普通的引用只能绑定左值, 一个右值引用只能绑定到右值, 也就是说如果我们写了下面的代码, 编译器会报一个错误.

```cpp
int&& t = n;    // Error: n is an lvalue
```

右值引用是一个有趣的组合, 虽然你只能绑定右值到右值引用, 但是你使用时它却是一个左值.

例如下面的代码:

```cpp
int&& t = 3;
++t;
```

通常情况下, 程序员不能修改右值. 但是通过右值引用, 获得了左值, 这样我们就能修改它的值了. 另外, 右值在通过右值引用绑定之后, 这个右值就不在可访问, 它使用了转移语义来代替拷贝操作.
 
定义一个接收右值引用参数的函数, 那么函数就会使用转移语义来替代拷贝操作. 但是这样会禁止接收左值参数.

```cpp
void foo(int&&);
foo(42);        // OK;
int n;
foo(n);         // Error: n is an lvalue
```

我们当然可以定义一个左值的版本来解决上面的问题:

```cpp
void bar(int&&);
void bar(const int&);
bar(42);             // calls bar(int&&)
int n;
bar(n);               // calls bar(const int&)
```

这个技术有个值得记住的地方是, 通过 const 引用和右值引用的两个重载版本, 你告诉了编译器哪个修改参数是安全的, 哪个不是. 此外, 编译器在编译期就决定了使用哪个, 从而避免了运行时的开销.


##Move 语义

右值引用的引入, 从语言层面提升了性能, 提高了内存与时间上的效率.
 
在 C++03 及之前的标准中, 临时对象 (称为右值 "R-values", 位于赋值运算符之右) 无法被改变, 在 C 中亦同 (且被视为无法和 const T& 做出区分). 虽然在某些情况下临时对象的确会被改变, 甚至也被视为是一个有用的漏洞.
 
C++11 增加一个新的非常数引用 (reference) 类型, 称作右值引用 (R-value reference), 标记为 `T&&`. 右值引用所引用的临时对象可以在该临时对象被初始化之后做修改, 这是为了允许 move 语义.
 
C++03 性能上被长期被诟病的其中之一, 就是其耗时且不必要的深度拷贝. 深度拷贝会发生在当对象是以传值的方式传递. 举例而言, `std::vector<T>` 是内部保存了 C-style 数组的一个包装, 如果一个 `std::vector<T>` 的临时对象被建构或是从函数返回, 要将其存储只能通过生成新的 `std::vector<T>`并且把该临时对象所有的数据复制进去. 该临时对象和其拥有的內存会被摧毁. (为了讨论上的方便，这里忽略返回值优化)
 
在 C++11 中, 一个 `std::vector` 的 "move 构造函数" 对某个 vector 的右值引用可以单纯地从右值复制其内部 C-style 数组的指针到新的 vector, 然后留下空的右值. 这个操作不需要数组的复制, 而且空的临时对象的析构也不会摧毁内存. 传回 vector 临时对象的函数不需要显式地传回 `std::vector<T>&&`. 如果 vector 没有 move 构造函数, 那么复制构造函数将被调用, 以 `const std::vector<T> &` 的正常形式. 如果它确实有 move 构造函数, 那么就会调用 move 构造函数, 这能够免除大幅的内存配置.
 
基于安全的理由, 具名的参数将永远不被认定为右值, 即使它是被如此声明的; 为了获得右值必须使用 `std::move<T>()`.

```cpp
bool is_r_value(int &&) { return true; }
bool is_r_value(const int &) { return false; }
 
void test(int && i)
{
    is_r_value(i); // i 为具名变量，即使被声明成右值也不會被认为是右值。
    is_r_value(std::move<int&>(i)); // 使用 std::move<T>() 取得右值。
}
```

由于右值引用的用语特性以及对于左值引用 (L-value references;regular references) 的某些用语修正, 右值引用允许开发者提供完美转发 (perfect function forwarding). 当与变长参数模板结合, 这项能力允许函数模板能够完美地转送引数给其他接受这些特定引数的函数. 最大的用处在于转发构造函数参数, 创造出能够自动为这些特定引数调用正确建构式的工厂函数 (factory function).


##参考

- [《C++0x漫谈》系列之：右值引用(或“move语意与完美转发”)(上)](http://blog.csdn.net/pongba/article/details/1684519)
- [维基百科C++11: 右值引用和 move 语义](https://zh.wikipedia.org/zh-cn/C%2B%2B11#.E5.8F.B3.E5.80.BC.E5.BC.95.E7.94.A8.E5.92.8C_move_.E8.AA.9E.E6.84.8F)
- [Stackoverflow：What are move semantics？](http://stackoverflow.com/questions/3106110/what-are-move-semantics/3109981)
- [标准文档提案：Move Semantics](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n2027.html#Move_Semantics)
