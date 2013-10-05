---
layout: post
title: "异步编程 async & await"
date: 2013-08-20 01:17
comments: true
categories: [async]
---

##使用异步编程

使用异步编程, 可以避免性能瓶颈和增强应用程序的总体响应能力. 但是编写异步程序在以前技术上比较复杂, 使得它们难以编写, 调试以及维护.

在 .NET Framework 4.5 中, 引入了简化的异步编程方式, 编译器帮我们实现了之前需要实现的异步操作, 程序员只需要关注自己的代码逻辑即可. 程序员可以更专注到业务逻辑当中.
 
##async & await
 
在 C# 中, 关键字 async 和 await 是异步编程的核心, 当使用这两个关键字的时候, 我们可以直接创建一个异步函数, 就像创建一个同步函数一样简单. 看下面的例子, 所有的东西看起来都是那么的熟悉.

```cpp 
// Three things to note in the signature: 
//  - The method has an async modifier.  
//  - The return type is Task or Task<T>. (See "Return Types" section.)
//    Here, it is Task<int> because the return statement returns an integer. 
//  - The method name ends in "Async."
async Task<int> AccessTheWebAsync()
{
    // You need to add a reference to System.Net.Http to declare client.
    HttpClient client = new HttpClient();
 
    // GetStringAsync returns a Task<string>. That means that when you await the 
    // task you'll get a string (urlContents).
    Task<string> getStringTask = client.GetStringAsync("http://msdn.microsoft.com");
 
    // You can do work here that doesn't rely on the string from GetStringAsync.
    DoIndependentWork();
 
    // The await operator suspends AccessTheWebAsync. 
    //  - AccessTheWebAsync can't continue until getStringTask is complete. 
    //  - Meanwhile, control returns to the caller of AccessTheWebAsync. 
    //  - Control resumes here when getStringTask is complete.  
    //  - The await operator then retrieves the string result from getStringTask. 
    string urlContents = await getStringTask;
 
    // The return statement specifies an integer result. 
    // Any methods that are awaiting AccessTheWebAsync retrieve the length value. 
    return urlContents.Length;
}
```

<!-- more -->
##async 方法中到底发生了什么？
 
在异步编程中, 最重要的需要了解的事情是, 方法与方法之间的执行流程是什么. 下面的图可以帮助我们理解执行流程.

![](https://qhphgq.bay.livefilestore.com/y2pPcdpEqnMPWdvKOr5yjswLE7F201mPiHRsjrk0PteSCWt9jdSWQxA3Tgkas4qOECf2vishbMkp4GnpFvE_ZQJcMwRppgT7hM0UrQxVevorwg/async.png)

图上的标记对应以下步骤

1. 事件处理程序调用并等待 AccessTheWebAsync 异步方法.
2. AccessTheWebAsync 创建 HttpClient 实例并调用GetStringAsync 异步方法下载网页内容, 作为字符串.
3. GetStringAsync 会造成挂起, 这时会发生一些事情. 可能它必须等待网页下载完成或者其他一些会阻塞的事情. 为了避免阻塞, GetStringAsync 会交出控制权给他的调用者, AccessTheWebAsync. AccessTheWebAsync 是返回一个模板参数是 string 的 Task 对象, 赋值给 getStringTask 变量. task 不会因为 GetStringAsync 的调用而阻塞当前进程, 而是当任务实际执行完成时, 再提交.
4. 由于 getStringTask 此时还没有 await, AccessTheWebAsync 可以继续执行一些不依赖 GetStringAsync 结果的一些操作. 这些工作在一个同步的叫做 DoIndependentWork 函数中执行.
5. DoIndependentWork 是一个同步函数, 当它的工作做完之后会将结果返回给调用者.
6. AccessTheWebAsync 做完了不依赖 getStringTask 的所有事情, AccessTheWekAsync 接下来需要计算 getStringTask 的结果, 但是此时函数还没有返回. 因此, AccessTheWebAsync 使用了 await 将当前的操作挂起, 然后将控制权移交到 AccessTheWebAsync 的调用者. AccessTheWebAsync 返回一个 task<int> 给调用者. 这个 task 给了外部一个承诺 (promise) 当计算完成时会返回给调用者一个整数, 这个整数就是下载的数据量的长度. *注意: 如果 GetStringAsync (以及 getStringTask) 在 AccessTheWebAsync await 之前就完成了, 那么控制权会仍然在 AccessTheWebAsync 中. 如果在不需要 await 时, 还继续进行代价很高的控制权转移的话, 是严重的浪费.* 在调用者中 (本例的 event handler), 这个过程会重复多次. 调用者可能会在 await 之前, 做一些其他的不依赖 AccessTheWebAsync 的操作, 或者直接 await. 当 event handler 执行到 await 语句时, 应用程序正在等待 AccessTheWebAsync, 同时 AccessTheWebAsync 正在等待 GetStringAsync.

7. GetStringAsync 完成工作并返回了字符串结果. 这个字符串不是直接由 GetStringAsync 调用返回的. (在第 3 步的时候我们就已经返回了一个 task). 取而代之, 这个字符串是存储在已完成的 task 中的, getStringTask. await 会得到 getStringTask 的结果. 赋值语句会将得到的结果赋值给 urlContents.
8. 当 AccessTheWebSync 得到了结果之后, 他就可以计算字符串的长度了. 然后 AccessTheWebAsync 的任务就全部完成了. 正在等待的 event handler 就可以恢复工作了.

##C++ 中的 async 以及 await

上面是一个 C# 的例子, C++11 中虽然对线程的支持作了相当多的工作, 但是异步编程方面还没有引入与 C# 中的 async 以及 await 类似的东西. 但标准委员会正在做这方面的工作, 提案名字叫做 Resumable Functions (n3650), 虽然 C++14 中不会引入这个特性 (这个特性应该是 C++1y 中的特性 (y应该等于7)), 但微软会很快的将该特性的实现引入到新版的 visual studio 中.
 
一个例子感受一下没有 async 以及 await 时我们是如何做异步编程的

```cpp
task<std::string> read(std::string file, std::string suffix) {
   return open(file)
   .then([=](std::istream fi) {
      auto ret = std::make_shared<std::string>();
      auto next =
         std::make_shared<std::function<task()>>(
      [=]{
         fi.read()
         .then([=](std::string chunk) {
            if( chunk.size() ) {
               *ret += chunk + suffix;
               return (*next)();
            }
            return *ret;
         });
      });
      return (*next)();
   });
}
```

看看上面的一堆代码，WTF，谁能快速的搞明白？？？？？
 
再看看下面清爽的代码吧…

```cpp
task<std::string> read( std::string file, std::string suffix ) __async {
   std::istream fi = __await open(file);
   std::string ret, chunk;
   while((chunk = __await fi.read()).size())
      ret += chunk + suffix;
   return ret;
}
```

是不是迫不及待的想要尝试了呢？
 
##参考

- [Asynchronous Programming with Async and Await (C# and Visual Basic)](http://msdn.microsoft.com/en-us/library/vstudio/hh191443.aspx)
- [The future of C++](http://channel9.msdn.com/Events/Build/2013/2-306)
- [Resumable Functions](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2013/n3650.pdf)
