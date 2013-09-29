---
layout: post
title: "使用 condition variable 实现生产者 消费者模型"
date: 2013-07-22 00:05
comments: true
categories: [C++]
---

##什么是生产者-消费者问题?

生产者消费者问题 (英语: Producer-consumer problem), 也称有限缓冲问题 (英语: Bounded-buffer problem), 是一个多线程同步问题的经典案例. 该问题描述了两个共享固定大小缓冲区的线程 —— 即所谓的 “生产者” 和 “消费者” —— 在实际运行时会发生的问题. 生产者的主要作用是生成一定量的数据放到缓冲区中, 然后重复此过程. 与此同时, 消费者也在缓冲区消耗这些数据. 该问题的关键就是要保证生产者不会在缓冲区满时加入数据, 消费者也不会在缓冲区中空时消耗数据.
 
要解决该问题, 就必须让生产者在缓冲区满时休眠 (要么干脆就放弃数据), 等到下次消费者消耗缓冲区中的数据的时候, 生产者才能被唤醒, 开始往缓冲区添加数据. 同样, 也可以让消费者在缓冲区空时进入休眠, 等到生产者往缓冲区添加数据之后, 再唤醒消费者. 通常采用进程间通信的方法解决该问题, 常用的方法有信号量等. 如果解决方法不够完善, 则容易出现死锁的情况. 出现死锁时, 两个线程都会陷入休眠, 等待对方唤醒自己. 该问题也能被推广到多个生产者和消费者的情形.


<!-- more -->
##使用 condition variable 实现生产者-消费者模型

```cpp
#include <condition_variable>
#include <future>
#include <iostream>
#include <mutex>
#include <sstream>
#include <vector>
 
std::mutex buffer_mutex;
std::condition_variable buffer_not_empty;
std::condition_variable buffer_not_full;
 
std::vector<int> buffer;
 
const std::size_t kBufferMaxSize = 20;
 
void print(std::ostream&& s)
{
  std::cout << s.rdbuf(); std::cout.flush(); s.clear();
}
 
void send(int i)
{
  std::unique_lock<std::mutex> lk(buffer_mutex);
  buffer_not_full.wait(lk, [&]{ return buffer.size() < kBufferMaxSize; });
  buffer.push_back(i);
  print(std::stringstream() << "produce: " << i << "\n");
  buffer_not_empty.notify_all();
}
 
void receive()
{
  std::unique_lock<std::mutex> lk(buffer_mutex);
  buffer_not_empty.wait(lk, [&]{ return !buffer.empty(); });
  int i = buffer.back();
  buffer.pop_back();
  print(std::stringstream() << "consume: " << i << "\n");
  buffer_not_full.notify_all();
}
 
int main(int argc, char* argv[])
{
  auto future_producer = std::async(std::launch::async, []{
    for (int i = 0;; ++i) {
      send(i);
    }
  });
  auto future_consumer = std::async(std::launch::async, []{
    while (true) {
      receive();
    }
  });
  future_producer.wait();
  future_consumer.wait();
  return 0;
}
```

##参考

- [wikipedia: Producer–consumer problem](http://en.wikipedia.org/wiki/Producer–consumer_problem)
- [cppreference.com: std::condition_variable](http://en.cppreference.com/w/cpp/thread/condition_variable)

