---
layout: post
title: "C++11中的原子操作 (atomic operation)"
date: 2013-09-02 23:34
comments: true
categories: [C++, C++11, multithread]
---

所谓的原子操作, 取的就是 “原子是最小的, 不可分割的最小个体” 的意义, 它表示在多个线程访问同一个全局资源的时候, 能够确保所有其他的线程都不在同一时间内访问相同的资源. 也就是他确保了在同一时刻只有唯一的线程对这个资源进行访问. 这有点类似互斥对象对共享资源的访问的保护, 但是原子操作更加接近底层, 因而效率更高.

在以往的 C++ 标准中并没有对原子操作进行规定, 我们往往是使用汇编语言, 或者是借助第三方的线程库, 例如 intel 的 pthread 来实现. 在新标准 C++11, 引入了原子操作的概念, 并通过这个新的头文件提供了多种原子操作数据类型, 例如, atomic_bool, atomic_int 等等, 如果我们在多个线程中对这些类型的共享资源进行操作, 编译器将保证这些操作都是原子性的, 也就是说, 确保任意时刻只有一个线程对这个资源进行访问, 编译器将保证, 多个线程访问这个共享资源的正确性. 从而避免了锁的使用, 提高了效率.

我们还是来看一个实际的例子. 假若我们要设计一个广告点击统计程序, 在服务器程序中, 使用多个线程模拟多个用户对广告的点击:

```cpp
#include <iostream>
#include <thread>
#include <time.h>
// 全局的结果数据
long total = 0;
// 点击函数
void click()
{
  for(int i = 0; i < 10000; ++i) {
    // 对全局数据进行无锁访问
    total += 1;
  }
}

int main(int argc, char* argv[])
{
  // 计时开始
  clock_t start = clock();
  // 创建100个线程模拟点击统计
  std::thread threads[100];
  for(int i = 0; i < 100; ++i) {
    std::thread t(&click);
    threads[i].swap(t);
  }
  for(int i=0; i<100; ++i) {
    threads[i].join();
  }
  // 计时结束
  clock_t finish = clock();
  // 输出结果
  std::cout << "result:" << total << std::endl;
  std::cout << "duration:" << finish -start << "ms" << std::endl;
  return 0;
}
```
<!-- more -->
执行结果如下:

> result: 955700
>
> duration: 49ms

从执行的结果来看, 这样的方法虽然非常快, 但是结果不正确.

很自然地, 我们会想到使用互斥对象来对全局共享资源的访问进行保护, 于是有了下面的实现:

```cpp
///
std::mutex m;
// 点击函数
void click()
{
  for(int i = 0; i < 10000; ++i) {
    // 对全局数据进行无锁访问
    std::unique_lock<std::mutex> lk(m);
    total += 1;
  }
}
///
```

运行结果如下:

> result: 1000000
>
> duration: 32350ms

互斥对象的使用, 保证了同一时刻只有唯一的一个线程对这个共享进行访问, 从执行的结果来看, 互斥对象保证了结果的正确性, 但是也有非常大的性能损失, 从刚才的 49ms 变成了现在的 32350ms. 这 TMD 差距也太大了.(感觉有些异常，差距没理由这么大的, 可能是线程太多, CPU 光忙着线程切换了 = =!)

如果是在 C++11 之前, 我们的解决方案也就到此为止了. 但是, C++ 对性能的追求是永无止境的, 他总是想尽一切办法榨干 CPU 的性能. 在 C++11 中, 实现了原子操作的数据类型 (atomic_bool, atomic_int, atomic_long 等等), 对于这些原子数据类型的共享资源的访问, 无需借助 mutex 等锁机制, 也能够实现对共享资源的正确访问.
 
将原本的 long 类型改为 std::atomic_long 类型即可

```cpp
//long total = 0;
 
std::atomic_long total;
``` 
 
运行结果如下:

> result: 1000000
>
> duration: 117ms

结果正确, 时间上仅仅是不采用任何保护机制时的 3 倍左右, 和使用 mutex 做同步的版本不知差了几个数量级了.

原子操作的实现跟普通数据类型类似, 但是它能够在保证结果正确的前提下, 提供比 mutex 等锁机制更好的性能, 如果我们要访问的共享资源可以用原子数据类型表示, 那么在多线程程序中使用这种新的等价数据类型, 是一个不错的选择. 

但值得注意的是, `std::atomic` 只能作用于基本类型以及 POD 类型上.

##参考

- [CppReference.com - atomic](http://en.cppreference.com/w/cpp/atomic/atomic)
- [C++ and Beyond 2012: Herb Sutter - atomic<> Weapons](http://channel9.msdn.com/Shows/Going+Deep/Cpp-and-Beyond-2012-Herb-Sutter-atomic-Weapons-1-of-2)

