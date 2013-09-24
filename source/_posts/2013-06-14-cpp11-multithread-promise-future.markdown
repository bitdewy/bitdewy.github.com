---
layout: post
title: "C++11 多线程, std::future & std::promise"
date: 2013-06-14 22:36
comments: true
categories: [C++, C++11, multithread]
---

C++11 中最让人高兴的新特性中线程库的支持一定榜上有名. C++11 中提供了 future 和 promise 来简化任务线程间的返回值操作; 同时为启动任务线程提供了 [packaged_task][packaged_task] 以方便操作.

##std::packaged_task

模板类 [std::packaged_task][packaged_task] 可以包装任何可调用的对象(函数, lambda 表达式, std::bind, 或其他函数对象), 以便异步调用, 调用结果保存在 [std::future][future] 中, 可以通过成员函数 get_future 访问. 需要注意的是, [std::packaged_task][packaged_task] 是不可拷贝的(move only).

示例代码

```cpp
#include <iostream>
#include <cmath>
#include <thread>
#include <future>
#include <functional>
 
void task_lambda()
{
    std::packaged_task<int(int,int)> task([](int a, int b) {
        return std::pow(a, b); 
    });
    std::future<int> result = task.get_future();
    task(2, 9);
    std::cout << "task_lambda:\t" << result.get() << '\n';
}
 
void task_bind()
{
    std::packaged_task<int()> task(std::bind(std::pow, 2, 11));
    std::future<int> result = task.get_future();
    task();
    std::cout << "task_bind:\t" << result.get() << '\n';
}
 
void task_thread()
{
    std::packaged_task<int(int,int)> task(std::pow);
    std::future<int> result = task.get_future();
    std::thread task_td(std::move(task), 2, 10);
    task_td.join();
    std::cout << "task_thread:\t" << result.get() << '\n';
}
 
int main()
{
    task_lambda();
    task_bind();
    task_thread();
}
```

<!-- more -->
仅仅有 packaged_task 还远远不够, 我们还需要更强大的 [future][future] 和 [promise][promise].

基本思路很简单: 当一个任务需要向父线程(启动它的线程)返回值时, 它把这个值放到 [promise][promise] 中. 之后, 这个返回值会出现在和此 [promise][promise] 关联的 [future][future] 中.于是父线程就能读到返回值. 更简单点的方法, 参看 [async()][async].

如果我们有一个 [future][future] f, 通过 get() 可以获得它的值:

```cpp
X v = f.get();  // if necessary wait for the value to get computed
```

如果它的返回值还没有到达, 调用线程会进行阻塞等待. 要是等啊等啊, 等到花儿也谢了的话, get() 会抛出异常的(从标准库或等待的线程那个线程中抛出).

如果我们不需要等待返回值(非阻塞方式), 可以简单询问一下 [future][future], 看返回值是否已经到达:

```cpp
if (f.wait_for(0))  {
    // there is a value to get()
    // do something
} else {
    // do something else
}
```

但是 [future][future] 最主要的用途是一个简单的获取返回值的方法: get().

[promise][promise] 的主要用途是提供一个 "put"（或"get"，随你）操作, 以和 [future][future] 的 get() 对应.

[promise][promise] 为 [future][future] 传递的结果类型有 2 种: 传一个普通值或者抛出一个异常

```cpp
try { 
    X res; 
    // compute a value for res
    p.set_value(res);
    
} catch (...) {   // oops: couldn't compute res 
    p.set_exception(std::current_exception()); 
}
```

到目前为止还不错, 不过我们如何匹配 [future][future] / [promise][promise] 对呢? 一个在我的线程, 另一个在别的啥线程中吗? 是这样: 既然 [future][future] 和 [promise][promise] 可以被到处移动(不是拷贝), 那么可能性就挺多的. 最普遍的情况是父子线程配对形式, 父线程用 [future][future] 获取子线程 [promise][promise] 返回的值. 在这种情况下, 使用 [async()][async] 是很优雅的方法.


##std::promise

模板类 [std::promise][promise] 提供一个存储设施, 当一个异步任务通过 [std::future][future] 来获取结果时, [std::promise][promise] 可以提供.


##std::future

模板类 [std::future][future] 提供了一种机制来访问异步操作的结果:
由 [std::async][async], [std::packaged_task][packaged_task], 或者 [std::promise][promise] 创建的异步操作, 可以提供一个 [std::future][future] 对象, 给创建者, 用来访问异步操作的结果.

异步操作的创建者, 可以使用各种不同的操作来查询, 等待, 或者从 [std::future][future] 中取到异步操作的结果. 如果异步操作还没有执行完的话, 这些操作有可能会阻塞.
当一个异步操作完成时可以通过更改(例如使用 `std::promise::set_value`) 共享状态(该状态保存在 [std::future][future] 中) 将结果返回给异步操作的创建者.

示例代码

```cpp
#include <iostream>
#include <future>
#include <thread>
int main()
{
    // future from a packaged_task
    std::packaged_task<int()> task([](){ return 7; }); // wrap the function
    std::future<int> f1 = task.get_future();  // get a future
    std::thread(std::move(task)).detach(); // launch on a thread
    // future from an async()
    std::future<int> f2 = std::async(std::launch::async, [](){ return 8; });
    // future from a promise
    std::promise<int> p;
    std::future<int> f3 = p.get_future();
    std::thread([](std::promise<int>& p){ p.set_value(9); },
                 std::ref(p)).detach();
    std::cout << "Waiting..." << std::flush;
    f1.wait();
   f2.wait();
    f3.wait();
    std::cout << "Done!\nResults are: "
              << f1.get() << ' ' << f2.get() << ' ' << f3.get() << '\n';
}
```

##参考

- [Cppreference：Thread support library](http://en.cppreference.com/w/cpp/thread)
- [wikipedia: future and promises](http://en.wikipedia.org/wiki/Futures_and_promises)

  [packaged_task]: http://en.cppreference.com/w/cpp/thread/packaged_task
  [future]: http://en.cppreference.com/w/cpp/thread/future
  [promise]: http://en.cppreference.com/w/cpp/thread/promise
  [async]: http://en.cppreference.com/w/cpp/thread/async
