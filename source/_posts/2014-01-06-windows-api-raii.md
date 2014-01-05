---
layout: post
title: "当 Windows API 遇上 RAII"
date: 2014-01-06 00:18
comments: true
categories: [C++]
---

##什么是 RAII (Resource Acquisition Is Initialization) ?

RAII (Resource Acquisition Is Initialization), 也称为"资源获取就是初始化", 是 C++ 语言的一种管理资源, 避免泄漏的惯用法. C++ 标准保证任何情况下, 已构造的对象最终会销毁, 即它的析构函数最终会被调用. 简单的说, RAII 的做法是使用一个对象, 在其构造时获取资源, 在对象生命期控制对资源的访问使之始终保持有效, 最后在对象析构的时候释放资源.

RAII 是保证代码异常安全的重要基础设施. RAII 的使用场景有很多, 如: C++11 中的智能指针, scope lock, scope exit 等等. (早在2000年，[Andrei Alexandrescu](http://erdani.com/) 就在DDJ杂志上发表了一篇文章，提出了这个叫做 [ScopeGuard](http://www.drdobbs.com/cpp/generic-change-the-way-you-write-excepti/184403758) 的设施)


##当 Windows API 遇上 RAII

Windows API 大多是 C 语言风格的函数和句柄, 或者是 COM 风格的接口, 这些用起来都不太方便, 需要进行一定的封装. 至于为什么要封装就不用多说了, 如果你想要异常安全, 想要不必在每个分支中都写清理代码的话, 你一定知道利用 RAII 封装的意义.

ATL 中有对 COM 接口的封装, 智能指针 `CComPtr`, `CComQIPtr` 解决了一遍遍的手工 `Release` 以及 `QueryInterface`. 但对于普通的 C 语言风格的函数和句柄呢? 难道还要一遍遍的 `CloseHandle` , `ReleaseDC`, `GlobalUnlock` 麽? 弱爆了.

借助 `ScopeGuard` 和 lambda 表达式(⊙_⊙)？ 可以是可以, 但是并不是所有的资源获取都会成功, 那么每次都要产生一个具名的 `ScopeGuard`, 在申请失败的时候调用 `Dismiss`, 取消清理的动作嘛? 像这样:

```cpp
Acquire Resource1
ScopeGuard release1([&] { /* Release Resource1 */ })
if (!Resource1) {
	release1.Dismiss();
	// throw exception or return or...
}

Acquire Resource2
ScopeGuard release2([&] { /* Release Resource2 */ })
if (!Resource2) {
	release2.Dismiss();
	// throw exception or return or...
}
```
<!-- more -->
这样如果连续申请多个资源 `ScopeGuard` 对象命名都会成为问题. 又或者是先判断资源是否申请成功, 然后再使用匿名的 `ScopeGuard` 来保证正确释放资源? 像这样:

```cpp
Acquire Resource1
if (!Resource1) {
	// throw exception or return or...
}
ON_SCOPE_EXIT([&] { /* Release Resource1 */ })


Acquire Resource2
if (!Resource2) {
	// throw exception or return or...
}
ON_SCOPE_EXIT([&] { /* Release Resource2 */ })

```

这样好是好, 可是割裂了申请与释放的代码, 而且每申请一个资源就要至少写上 3 行或以上结构类似代码(可以考虑宏了) ?

当然, 我们是有追求的, 我们还需要更厉害的设施. 也许 `unique_ptr` 可以给我提供一些思路. 我们需要一个基本的类型, 也许是 `HANDLE`, 也许是 `HINTERNET` 等等, 同时我们还需要一个清理函数. 再加上一个资源是否可用的接口即可. 为了避免过多的模板参数, 我们把清理函数以及不可用资源封装到 `Traits` 类中, 下面是一个例子, 可以很好的完成我们的需求. 另外仿照 `unique_ptr` 加了一些 move 语义, 转移构造等东西. 下面的代码只实现了 `HANDLE` 的特化版本, 相信 `HINTERNET` 的版本, 大家写起来应该也是毫无压力了. 只需要写 `Traits` 类即可.

```cpp
template <typename T, typename Traits>
class unique_handle {
  struct bool_struct { int member; };
  using bool_type = int bool_struct::*;

public:
  explicit unique_handle(T value = Traits::invalid())
    : value_(value) {}
  unique_handle(unique_handle&& other)
    : value_(other.release()) {}
  unique_handle& operator=(unique_handle&& other) {
    reset(other.release());
    return *this;
  }
  ~unique_handle() { close(); }
  T get() { return value_; }
  bool reset(T value = Traits::invalid()) {
    if (value_ != value) {
      close();
      value_ = value;
    }
    return *this;
  }
  T release() {
    auto value = value_;
    value_ = Traits::invalid();
    return value;
  }
  operator bool_type() {
    return Traits::invalid() != value_ ? &bool_struct::member : nullptr;
  }
private:
  unique_handle(const unique_handle&);
  unique_handle& operator=(const unique_handle&);
  bool operator==(const unique_handle&);
  bool operator!=(const unique_handle&);
  void close() {
    if (*this) {
      Traits::close(value_);
    }
  }
  T value_;
};

struct handle_traits {
  static HANDLE invalid() {
    return nullptr;
  }
  static void close(HANDLE handle) {
    CloseHandle(handle);
  }
};

typedef unique_handle<HANDLE, handle_traits> handle;
```

使用起来应该是这样的:

```cpp
unique_handle<SOCKET, socket_traits> socket;
unique_handle<HANDLE, handle_traits> event;
 
if (socket && event) {} // Are both valid?
if (!event) {} // Is event invalid?
int i = socket; // Compiler error!
if (socket == event) {} // Compiler error!
```

`int i = socket;` 这一句很有意思, 我们为了让它能够编译失败, 费了不少功夫. 内部类 `bool_struct` 就是完全为它而准备. 这也是为什么我们不直接提供 `operator bool` 的原因. 私有化 `operator==` 和 `operator!=` 是为了禁止两个资源进行比较. 而使用内部类成员指针就是为了提供 `operator bool` 类似功能的同时, 避免它能够提升为 `int` 等类型. 当然如果我们直接提供一个 `is_valid` 成员函数, 而不提供隐身转换, 那么就不会有这么多的问题了.

看起来还差错误处理的内容, 不过都到这个份上了, 错误处理就不是问题了吧. 我们可以写各种 `check` 函数的重载版本, 当 `check` 失败时抛出异常. 这样就大功告成了.

##参考
- wikipedia: [Resource Acquisition Is Initialization](http://en.wikipedia.org/wiki/Resource_Acquisition_Is_Initialization)
- [C++11（及现代C++风格）和快速迭代式开发](http://mindhacks.cn/2012/08/27/modern-cpp-practices/)
- [C++ and the Windows API](http://msdn.microsoft.com/en-us/magazine/hh288076.aspx)
