---
layout: post
title: "异常与多线程"
date: 2013-09-23 23:30
comments: true
categories: [exception, multithread]
---

C++ 的异常机制为程序员提供了一种处理错误的方式, 使程序员可以用更自然的方式处理错误.(但 C++ 的异常存在各种坑, 参考《More Effective C++》Item 13: Catch exceptions by reference).

有些时候, 我们不得不使用多线程来提高工作的执行效率, 但由于异常都是基于线程的, 如何在线程间传递异常, 就成为了一个问题. C++11 之前异常是无法在线程之前传递的, C++11 在这方面提供了支持.
 
##使用 C++11 中的异常相关内容实现在线程间传递异常
 
相关类如下:

`std::exception_ptr`

- 一个智能指针类型的异常对象, 支持 bool 操作符, 可以存储空异常 (null) 对象. 也支持比较 (operator==).
 
`std::current_exception`

- 当在异常处理块中 (catch块) 调用时, 会捕获当前异常对象并由被捕获的对象创建一个 `std::exception_ptr` 对象并返回.
 
`std::make_exception_ptr`

- 接受一个 Exception 对象参数, 并由这个异常对象创建一个 `std::exception_ptr` 对象
 
`std::rethrow_exception`

- 接受一个 `std::exception_ptr` 参数, 重新抛出这个异常
 
(*以上的一切, 很大一部分都是因为 C++ 的值传递特性引起的一系列不适!!!*, !!(╯' - ')╯︵ ┻━┻)

<!-- more -->
示例代码

```cpp
#include <future>
#include <exception>
#include <iostream>
 
int calc(std::exception_ptr& eptr)
{
  try {
    std::string s;
    s.at(0);  // throw out of range exception
    eptr = std::make_exception_ptr(std::exception()); // if no exception use default
  } catch (...) {
    eptr = std::current_exception();
  }
  return 0;
}
 
int main()
{
  std::exception_ptr eptr;
  auto f = std::async(std::launch::async, std::bind(calc, std::ref(eptr)));
  try {
    int i = f.get();
    if (eptr) {
      std::rethrow_exception(eptr);
    }
    std::cout << i << std::endl;
  } catch (const std::exception& e) {
    std::cout << e.what() << std::endl;
  }
  return 0;
}
```

如代码所示:

函数 calc 在子线程中工作, 并有可能抛出异常. 主线程中, 等待子线程返回结果, 并在返回后检测是否有异常, 如果有异常, 则重新抛出异常. 主线程将上述工作都放到 try 块中, 如果有异常则打印. 这样, 子线程 calc 函数的异常就被成功的转移到了主线程中.
 
##参考

- 《More Effective C++》Item 13: Catch exceptions by reference
- Cppreference.com [http://en.cppreference.com/w/cpp/error](http://en.cppreference.com/w/cpp/error)
