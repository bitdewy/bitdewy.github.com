---
layout: post
title: "[译]Threads, Events and QObjects"
date: 2012-05-28 23:53
comments: true
categories: [C++, Qt] 
---

前言: [Qt wiki][wiki] 中这篇文章3月份再次更新，文章对 [QThread][] 的用法，使用场景，有很好的论述，可以作为 Qt 多线程编程的使用指南，原文在[这里][00]，原作者 [peppe][01] 开的讨论贴在[这里][02]。

原文以姓名标识-相同方式分享 2.5 通用版发布

[Creative Commons Attribution-ShareAlike 2.5 Generic][ccas]

  [wiki]: http://qt-project.org/wiki/
  [QThread]: http://doc.qt.nokia.com/latest/qthread.html
  [00]: http://qt-project.org/wiki/Threads_Events_QObjects
  [01]: http://qt-project.org/member/5319
  [02]: http://developer.qt.nokia.com/forums/viewthread/2423/
  [ccas]: http://creativecommons.org/licenses/by-sa/2.5/


##背景
在 [#qt IRC channel][10] [irc.freenode.net] 中，讨论最多的话题之一就是多线程。很多同学选择了多线程并行编程，然后……呃，掉进了并行编程的无尽的陷阱中。

由于缺乏 Qt 多线程编程经验（尤其是结合Qt 信号槽机制的异步网络编程）加上一些现有的其他语言（工具）的使用经验，导致在使用 Qt 时，一些同学有朝自己脚开枪的行为。Qt 的多线程支持是一把双刃剑：虽然 Qt 的多线程支持使得多线程编程变得简单，但同时也引入了一些其他特性（尤其是与 [QObject][] 的交互），这些特性需要特别小心。

本文的目的不是教你如何使用多线程，加锁、并行、扩展性，这不是本文的重点，而且这些问题已经有非常多的讨论，可以参考[这里][11] [doc.qt.nokia.com] 的推荐。本文作为 Qt 多线程的指南，目的是帮助开发者避免常见的陷阱，开发出更健壮的程序。

  [10]: irc://irc.freenode.net/#qt
  [11]: http://qt-project.org/doc/qt-4.8/threads.html#recommended-reading
  [QObject]: http://doc.qt.nokia.com/latest/qobject.html

<!-- more -->
###知识背景
本文不是介绍多线程编程的文章，继续阅读下面的内容你需要以下的知识背景：
 
- C++ 基础（强烈推荐，其他语言亦可）
- Qt 基础：QObject，信号槽，事件处理
- 什么是线程，以及一个线程和其他线程、进程和操作系统之间的关系
- 在主流的操作系统上，如何启动和停止一个线程，如何等待线程结束
- 如何使用互斥量（mutex），信号量（semaphore），条件等待（wait condition）创建线程安全/可重入的函数，结构和类。

本文中使用 Qt 的[名词定义][13] [doc.qt.nokia.com]：

  [13]: http://doc.qt.nokia.com/latest/threads-reentrancy.html

- **可重入** 如果多个线程同时访问某个类的（多个）对象且一个对象同时只有一个线程访问，是安全的，那么这个类是可重入的。如果多个线程同时调用一个函数且只访问该线程可见的数据，是安全的，那么这个函数是可重入的。换句话说，访问这些对象/共享数据时，必须通过外部加锁机制来实现串行访问，保证安全。
- **线程安全** 如果多个线程同时访问某个类的对象是安全的，那么这个类是线程安全的。如果多个线程同时调用一个函数（即使访问了共享数据）是安全的，那么这个函数时线程安全的。


##事件与事件循环
作为一个事件驱动的系统，事件和事件分发在 Qt 的架构中扮演着核心角色。本文不会全面覆盖这个主题；我们主要阐述和线程相关的一些概念（有关 Qt 事件系统的文章，请看[这里][20]，还有[这里][21]）。

  [20]: http://doc.qt.nokia.com/latest/eventsandfilters.html
  [21]: http://doc.qt.nokia.com/qq/qq11-events.html

在 Qt 中，一个事件是一个对象，它表示一个有趣的事情发生了；信号(signal)和事件(event)的主要区别在于：在我们的程序中事件的目标是确定的对象（这个对象决定如何处理该事件），但信号可以发到“任何地方”。从代码级别来讲，所有的事件对象都是 [QEvent][]  [doc.qt.nokia.com] 的子类，所有继承自 [QObject][] 的类都可以重写 `QObject::event()` 虚函数，来作为事件的目标处理者。

  [QEvent]: http://doc.qt.nokia.com/latest/qevent.html

事件即可以来自应用程序内部，也可以来自外部；例如：

- QKeyEvent 和 QMouseEvent 对象代表鼠标、键盘的交互，这些事件来自于窗口管理器。
- QTimerEvent 对象会在计时器超时的时候发送给另一个 [QObject][]，这些事件（通常）来自于操作系统。
- QChildEvent 对象会在添加或删除一个 child 时，发送给另一个 [QObject][]，这些事件来自于你的程序中。

关于事件，有一个很重要的事情，那就是事件不会一产生就发送给需要处理这个事件的对象；而是放到事件队列中，然后再发送。事件分发器会循环处理事件队列，把每个在队列中的事件分发给相应的对象，因此又叫做事件循环。从理论上讲，事件循环看起来是这样的：

``` cpp
while (is_active)
{
    while (!event_queue_is_empty)
        dispatch_next_event();
 
    wait_for_more_events();
}
```

在 Qt 的使用中，通过调用 `QCoreApplication::exec()` 进入 Qt 的主事件循环；这个函数会阻塞，直到调用 `QCoreApplication::exit()` 或 `QCoreApplication::quit()`，结束事件循环。

函数 "wait_for_more_events()" 会阻塞（不是忙等）直到有事件产生。稍加考虑我们就会发现，在这时事件一定是从外部产生的（事件分发器已经结束并且也没有新的事件在事件队列中等待分发）。因此，事件循环可以在以下几种情况下被唤醒：

- 窗口管理器（键盘/鼠标点击，和窗口的交互，等）
- 套接字（sockets）（数据可读、可写、有新连接，等）
- 定时器（定时器超时）
- 从其他线程发送来的事件（稍后讨论）

在 Unix-like 系统中，窗口管理器的活动（例如 X11）是通过套接字（socket）（Unix Domain or TCP/IP）通知给应用程序的，因为客户端是通过套接字和 X Server 通信的。如果我们使用内部的 socketpair(2) 来实现跨线程的消息发送，那么我们要做的就是通过某些活动唤醒事件循环：

- 套接字（socket）
- 定时器

系统调用 select(2) 是这么工作的：它监听着一个活动的文件描述符的集合，如果一段时间（可配置超时事件）内都没有活动那么它就会超时。Qt 所需要做的就是把 select 返回的结果转化为一个 [QEvent][] 对象（子类对象）然后把它放入事件队列中。现在你应该知道消息循环内部事怎么回事儿了吧。


###哪些东西需要事件循环
下面不是完整的列表，不过稍微思考一下，你就能猜出那些类需要消息循环了。

- **Widget 绘图（painting）和交互**：当接收到 QPaintEvent 对象时，函数 `QWidget::paintEvent()` 会被调用，QPaintEvent 对象的产生，有可能是调用 `QWidget::update()` (应用程序内部调用) 函数，或者来自窗口管理器（例如：把一个隐藏的窗口显示出来）。其他类型的交互（鼠标、键盘，等）也是一样的：这些事件都需要一个事件循环来分发事件。
- **定时器**：简单说，当 `select(2)` 或类似的调用超时的时候，定时器超时事件被触发，因此你需要事件循换来处理这些调用。
- **网络通信**：所有 low-level 的 Qt 网络通信类（QTcpSocket, QUdpSocket, QTcpServer，等）都设计为异步的。当调用 `read()` 函数时，它们仅仅返回当前可用的数据，当调用 `write()` 函数时，它们会安排稍后再写。仅仅当程序返回事件循环的时候，读/写操作才真正发生。注意虽然提供有同步的方法（那些以 waitFor* 命名的函数），但是它们并不好用，因为在等待的同时它们阻塞了事件循换。像 QNetworkAccessManager 这样的 high-level 类，同样需要事件循换，但不提供任何同步调用的接口。

###阻塞事件循环
在讨论为什么我们**不应该阻塞**事件循环之前，先说明一下“阻塞”的含义是什么。想像一下，有一个在点击时可以发送信号的按钮，信号绑定到我们的工作类对象的一个槽函数上，这个槽函数会做很多工作。当你点击按钮时，函数调用栈看起来应该像下面这样（栈底在上）：

``` cpp
main(int, char **)
QApplication::exec()
[…]
QWidget::event(QEvent *)
Button::mousePressEvent(QMouseEvent *)
Button::clicked()
[…]
Worker::doWork()
```

在 main() 函数中，我们通过调用 `QApplication::exec()` （第2行） 启动了一个事件循换。窗口管理器发送一个鼠标点击的事件，Qt 内核会得到这个事件，然后转化为一个 QMouseEvent 对象，通过 `QApplication::notify()`（此处没有列出）函数发送给 widget 的 `event()` 函数（第4行）。如果按钮没有重写 `event()` 函数，那么它的基类（QWidget）实现的 `event()` 函数会被调用。`QWidget::event()`检测到鼠标点击事件，然后调用相应的事件处理函数，就是上面代码中的 `Button::mousePressEvent()`（第5行）函数。我们重写了这个函数，让它发送一个 `Button::clicked()` 信号（第6行），这个信号会调用 Worker 类对象的槽函数 `Worker::doWork()` （第8行）。

当 Worker 对象正在忙于工作的时候，事件循环在做什么？我们也许已经猜到了：什么也不做了！事件循环分发了鼠标点击事件然后等待，等待事件处理者返回。我们**阻塞了事件循环**，这意味在槽函数 `doWork()` 返回之前，不会再有事件被分发出去，事件会不断进入事件队列而不能得到及时的处理。
当事件分发被卡住的时候，**窗口不会刷新**（QPaintEvent 对象在事件队列中），**不能响应其他的交互行为**（和前面的原因一样），**定时器超时事件不会触发**、**网络通信变慢然后停止**。此外，很多窗口管理器会检测到你的程序不再处理事件，而提示**程序无响应**。这就是为什么迅速的处理事件然后返回事件循环如此重要的原因。


###强制分发事件
那么，如果有一个耗时的任务同时我们又不想阻塞消事件循环，这时该如何去做？一个可能的回答是：把这个耗时的任务移动到其他的线程中，下一节中我们可以看到如何做。我们还有一个可选的办法，那就是在我们耗时的任务中通过调用 `QCoreApplication::processEvents()` 来手动强制跑起事件循环。`QCoreApplication::processEvents()` 会处理所有队列上的事件然后返回。
另一个可选的方案，我们可以利用 [QEventLoop][] [doc.qt.nokia.com] 强制再加入一个事件循环。通过调用 `QEventLoop::exec()` 函数，我们加入一个事件循环，然后连接一个信号到  `QEventLoop::quit()` 槽函数上，来让循环退出。例如：

  [QEventLoop]: http://doc.qt.nokia.com/latest/qeventloop.html

``` cpp
QNetworkAccessManager qnam;
QNetworkReply *reply = qnam.get(QNetworkRequest(QUrl(...)));
QEventLoop loop;
QObject::connect(reply, SIGNAL(finished()), &loop, SLOT(quit()));
loop.exec();
/* reply has finished, use it */
```

QNetworkReply 不提供阻塞的接口，同时需要一个事件循环。我们进入了一个局部的 QEventLoop，当 reply 发出 finished 信号时，这个事件循环就结束了。
通过“其他路径”重入事件循环时需要特别小心：这可能导致不期望的递归！回到刚才的按钮例子中。如果我们在槽函数 `doWork()` 中调用 `QCoreApplication::processEvents()` ，同时用户再次点击了按钮，这个槽函数 `doWork()` 会**再一次**被调用：

``` cpp
main(int, char **)
QApplication::exec()
[…]
QWidget::event(QEvent *)
Button::mousePressEvent(QMouseEvent *)
Button::clicked()
[…]
Worker::doWork() // first, inner invocation
QCoreApplication::processEvents() // we manually dispatch events and…
[…]
QWidget::event(QEvent * ) // another mouse click is sent to the Button…
Button::mousePressEvent(QMouseEvent *)
Button::clicked() // which emits clicked() again…
[…]
Worker::doWork() // DANG! we’ve recursed into our slot.
```

一个快速简单的规避办法是给 `QCoreApplication::processEvents()` 传入一个参数 `QEventLoop::ExcludeUserInputEvents`，它会告诉事件循环不要分发任何用户输入的事件（这些事件会停留在队列中）。
幸运的是，同样的问题**不会**出现在**删除事件**中（调用 `QObject::deleteLater()` 会发送该事件到事件队列中）。事实上，Qt 使用了特别的办法来处理它，当事件循环比 deleteLater 调用发生的事件循环更外层时，删除事件才会被处理。例如：

``` cpp
QObject *object = new QObject;
object->deleteLater();
QDialog dialog;
dialog.exec();
```

这不会导致 object 空悬指针（`QDialog::exec()` 中的事件循环，比 deleteLater 调用发生的地方层次更深）。同样的事情也会发生在 QEventLoop 启动的事件循环中。我只发现过一个例外（在 Qt 4.7.3 中），如果在没有任何事件循环的时候调用了 deleteLater，那么第一个启动的事件循环会处理这个消息，删除该对象。这是很合理的，因为 Qt 知道不会有任何会执行删除动作的“外层”循环，因此会立即删除该对象。


##Qt线程类
Qt 支持多线程已经很多年（2000 年9月22日发布的 Qt 2.2 引入了 QThread 类），4.0 版本在所有平台上都默认开启多线程支持（多线程支持是可以关闭的，更多细节看[这里][30][doc.qt.nokia.com]）。Qt 现在提供了很多类来实现多线程；下面就来看一下。

  [30]: http://doc.qt.nokia.com/latest/fine-tuning-features.html


###QThread
[QThread][] [doc.qt.nokia.com] 是 Qt 中多线程支持的核心的 low-level 类。一个 QThread 对象表示一个执行的线程。由于 Qt 的跨平台特性，[QThread][] 设法隐藏了不同操作系统在线程操作中的所有平台相关的代码。

为了使用 [Qthread][] 在一个线程中执行代码，我们继承 [QThread][] 然后重写 `QThread::run()` 函数：

``` cpp
class Thread : public QThread {
protected:
    void run() {
        /* your thread implementation goes here */
    }
};
```

然后这么使用

``` cpp
Thread *t = new Thread;
t->start(); // start(), not run()!
```

来启动一个新的线程。注意，从 Qt 4.4 开始，QThread 不再是抽象类，现在虚函数 `QThread::run()` 有了调用 `QThread::exec()`的默认实现；它会启动线程自己的事件循环（稍后详细说明）。


###QRunnable 和 QThreadPool
[QRunnable][] [doc.qt.nokia.com] 是一个轻量级的抽象类，它可以在另一个线程中启动一个任务，适用于“运行完就丢掉”这种情况。实现这个功能，我们需要做的就是继承 [QRunnable][] 然后实现纯虚函数 `run()`:

  [QRunnable]: http://doc.qt.nokia.com/latest/qrunnable.html

``` cpp
class Task : public QRunnable {
public:
    void run() {
        /* your runnable implementation goes here */
    }
}
```

我们使用 [QThreadPool][] [doc.qt.nokia.com] 类，它管理着一个线程池，来真正运行一个 [QRunnable][] 对象。当调用 `QThreadPool::start(runnable)` 时，我们将 [QRunnable][] 对象放入 [QThreadPool][] 的执行队列中；当线程可用时，[QRunnable][] 对像会启动，然后在线程中执行。所有的 Qt 应用程序都有一个全局的线程池，可以通过调用  `QThreadPool::globalInstance()` 来获得，但是也可以创建一个私有的 [QThreadPool][] 对象来显式的管理。

注意，[QRunnable][] 不是一个 [QObject][]，因此没有 [QObject][] 内建的和其他一些组建通信的机制；你不得不使用 low-level 线程原语手工处理（例如用互斥量保护队列来收集结果等）。


###QtConcurrent
[QtConcurrent][] [doc.qt.nokia.com] 是 high-level API，在 [QThreadPool][] 基础上构建而成，它可以应用在大部分常用的并行计算范式中：[map][] [en.wikipedia.org]), [reduce][] [en.wikipedia.org]), 和 [filter][] [en.wikipedia.org])；它同时提供 `QtConcurrent::run()`方法，可以简单的在另一个线程中启动一个函数。

  [QtConcurrent]: http://doc.qt.nokia.com/latest/threads-qtconcurrent.html
  [QThreadPool]: http://doc.qt.nokia.com/latest/qthreadpool.html
  [map]: http://en.wikipedia.org/wiki/Map_(higher-order_function
  [reduce]: http://en.wikipedia.org/wiki/Fold_(higher-order_function
  [filter]: http://en.wikipedia.org/wiki/Filter_(higher-order_function

与 [QThread][] 和 [QRunnable][] 不同，[QtConcurrent][] 不需要我们使用 low-level 的同步原语：所有 [QtConcurrent][] 函数返回一个 [QFuture][] [doc.qt.nokia.com] 对象，它可以用来查询计算状态（进展），暂停/恢复/取消计算，同时它也包含计算的结果。[QFutureWatcher][] [doc.qt.nokia.com] 类可以用来监测 [QFuture][] 的进展，也可以通过信号槽来和 [QFuture][] 交互（注意，[QFuture][] 作为一个值语义的类，没有继承自 [QObject][]）。

  [QFuture]: http://doc.qt.nokia.com/latest/qfuture.html
  [QFutureWatcher]: http://doc.qt.nokia.com/latest/qfuturewatcher.html

 **特性对比**

    \                        QThread  QRunnable  QtConcurrent
    high level 接口           n        n          y
    面向任务                   n        y          y
    内建支持暂停、恢复、取消      n        n          y
    支持优先级                 y        n          n
    可以运行消息循环            y        n          n

QtConcurrent::run 是个例外，因为它是使用 [QRunnable][] 实现的，所以带有 [QRunnable][] 的特性。


##线程和QObject

###每个线程一个事件循环
到现在为止，我们已经讨论过“事件循环”，但讨论的仅仅是在一个 Qt 应用程序中只有一个事件循环的情况。但不是下面这种情况：QThread 对象可以启动一个自己线程中的事件循环。因此，我们把在 `main()` 函数中通过调用 `QCoreApplication::exec()`（该函数只能在主线程中调用）启动的事件循环叫做**主事件循环**。它也叫做 **GUI 线程**，因为 UI 相关的操作只能（应该）在该线程中执行。一个 [QThread][] 局部事件循环可以通过调用 `QThread::exec()` 来启动（在 run() 函数中）:

``` cpp
class Thread : public QThread {
protected:
    void run() {
        /* ... initialize ... */
 
        exec();
    }
};
```

上面我们提到，从 Qt 4.4 开始，`QThread::run()` 不再是一个纯虚函数，而是默认调用 `QThread::exec()`。和 QCoreApplication 一样，[QThread][] 也有 `QThread::quit()` 和 `QThread::exit()` 函数，来停止事件循环。

一个线程的事件循环为所有在这个线程中的 [QObject][] 对象分发事件；默认的，它包括所有在这个线程中创建的对象，或者从其他线程中移过来的对象（接下来详细说明）。同时，一个 [QObject][] 对象的线程相关性是确定的，也就是说这个对象生存在这个线程中。这个适用于在 [QThread][] 对象的构造函数中创建的对象：

``` cpp
class MyThread : public QThread
{
public:
    MyThread()
    {
        otherObj = new QObject;
    }    
 
private:
    QObject obj;
    QObject *otherObj;
    QScopedPointer<QObject> yetAnotherObj;
};
```

在创建一个 MyThread 对象之后，obj，otherObj，yetAnotherObj 的线程相关性如何？我们必须看看创建这些对象的线程：它是运行 MyThread 构造函数的线程。因此，所有这三个对象都不属于 MyThread 线程，而是创建了 MyThread 对象的线程（MyThread 对象也属于该线程）。

![](http://doc.qt.nokia.com/4.7/images/threadsandobjects.png)

我们可以使用线程安全的 `QCoreApplication::postEvent()` 函数来给对象发送事件。它会把事件放入该对象所在事件循环的事件队列中；因此，只有这个线程有事件循环，事件才会被分发。

理解 [QObject][] 和它的子类**不是线程安全**的（虽然它是可重入的）这非常重要；由于它不是线程安全的，所以你不能同时在多个线程中同时访问同一个 [QObject][] 对象，除非你自己串行化了所有对这些内部数据的访问（比如使用了互斥量来保护内部数据）。记住当你从其他线程访问 [QObject][] 对象时，这个对象有可能正在处理它所在的事件循环分发给它的事件。同样的，你也不能从另一个线程中删除一个 [QObject][] 对象，而必须使用 `QObject::deleteLater()` 函数，它会发送一个事件到对象所在线程中，然后在该线程中删除对象。

此外，QWidget 和它的所有子类，还有其他的 UI 相关类（非 [QObject][] 子类，比如 QPixmap）还是**不可重入**的：他们仅仅可以在 UI 线程中使用。

我们可以通过调用 `QObject::moveToThread()` 来改变 [QObject][] 对象和线程之前的关系，它会改变对象本身以及它的孩子与线程之前的关系。由于 [QObject][] 不是线程安全的，所以我们必须在它所在的线程中使用；也就是说，你仅仅可以在他们所处的线程中把它移动到另一个线程**去**，而不能从其他线程中把它从所在的线程中移动过**来**。而且，Qt 要求一个 [QObject][] 对象的孩子必须和他的父亲在同一个线程中，也就是说：

- 如果一个对象有父亲，那么你不能使用 `QObject::moveToThread()` 把它移动到其他线程
- 你不能在 [QThread][] 类中以 [QThread][] 为父亲创建对象

``` cpp
class Thread : public QThread {
    void run() {
        QObject *obj = new QObject(this); // WRONG!!!
    }
};
```

这是因为 [QThread][] **对象所在的线程是另外的线程**，即 [QThread][] 对象所在的线程是创建它的线程。

Qt 要求所有在线程中的对象必须在线程结束之前销毁；利用 `QThread::run()` 函数，在该函数中仅创建栈上的对象，这一点可以很容易的做到。


###跨线程信号槽
有了这些前提，我们如何调用另一个线程中 [QObject][] 对象的函数？ Qt 提供了一个非常漂亮和干净的解决方案：我们发送一个事件到线程的事件队列中，事件的处理，将调用我们感兴趣的函数（当然这个线程需要启动一个事件循环）。该设施围绕 Qt 的元对象编译器（MOC）提供的方法内省而构建：因此，信号，槽，函数，只要使用了 `Q_INVOKABLE` 宏，那么就可以从另外的线程调用它。

`QMetaObject::invokeMethod()` 静态方法为我们实现了这个功能：

``` cpp
QMetaObject::invokeMethod(object, "methodName",
                          Qt::QueuedConnection,
                          Q_ARG(type1, arg1),
                          Q_ARG(type2, arg2));
```

注意，由于参数需要在消息传递时拷贝，这些类型的参数需要提供公有的构造函数，析构函数和拷贝构造函数，而且要使用 `qRegisterMetaType()` 函数将类型注册到 Qt 类型系统中。

跨线程的信号槽工作方式是类似的。当我们将信号和曹连接时，`QObject::connect` 函数的第5个参数可以指定连接的类型：

- **direct connection**：意思是槽函数会在信号发送的线程中直接被调用
- **queued connection**：意思是事件会发送到接收者所在线程的消息队列中，消息循环会稍后处理该事件然后调用槽函数
- **blocking queued connection**：和 queued connection 类似，但是发送线程会阻塞，直到接收者所在线程的事件循环处理了该事件，调用了槽函数之后，才会返回

在任何情况下，记住发送者所在的线程一点都不重要！在自动连接的情况下，Qt 会检查信号调用的线程，然后与接收者所在线程比较，然后决定使用哪种连接类型。特别的，[Threads and QObjects][40] [doc.qt.nokia.com] \(4.7.1\) 在下面的情况下是**错误的**：

  [40]: http://doc.qt.nokia.com/4.7/threads-qobject.html

*自动连接（默认值），如果发送者和接收者在同一线程它和直接连接（direct connection）的行为是一样的；如果发送者和接收者在不同的线程它和队列连接（queued connection）的行为是一样的。*

因为发送者所在的线程和无关紧要的。例如：

``` cpp
class Thread : public QThread
{
    Q_OBJECT
 
signals:
    void aSignal();
 
protected:
    void run() {
        emit aSignal();
    }
};
 
/* ... */
Thread thread;
Object obj;
QObject::connect(&thread, SIGNAL(aSignal()), &obj, SLOT(aSlot()));
thread.start();
```

信号 aSignal() 会在一个新的线程中发送（Thread 对象创建的线程）；因为这不是 Object 对象所在的线程（但这时，Object 对象与 Thread 对象在同一个线程中，再次强调，发送者所在线程是无关紧要的），这时将使用 queued connection。

另一个常见的陷阱：

``` cpp
class Thread : public QThread
{
    Q_OBJECT
 
slots:
    void aSlot() {
        /* ... */
    }
 
protected:
    void run() {
        /* ... */
    }
};
 
/* ... */
Thread thread;
Object obj;
QObject::connect(&obj, SIGNAL(aSignal()), &thread, SLOT(aSlot()));
thread.start();
obj.emitSignal();
```

当“obj” 发送 aSignal() 信号时，将会使用哪种连接类型？你应该已经猜到了：direct connection。这是因为 Thread 对象所在线程就是信号发送的线程。在槽函数 aSlot() 中，我们可能访问 Thread 类的成员，而同时 run() 函数可能也在访问，它们会同时进行：这是完美的灾难配方。

另一个例子，或许也是最重要的一个：

``` cpp
class Thread : public QThread
{
    Q_OBJECT
 
slots:
    void aSlot() {
        /* ... */
    }
 
protected:
    void run() {
        QObject *obj = new Object;
        connect(obj, SIGNAL(aSignal()), this, SLOT(aSlot()));
        /* ... */
    }
};
```

在上面的情形中，连接类型是 queued connection，因此你需要在 Thread 对象所在线程启动一个事件循环。

下面是一个你经常可以在论坛、博客或其他地方看到的解决方案。那就是在 Thread 的构造函数中增加一个 `moveToThread(this)` 函数：

``` cpp
class Thread : public QThread {
    Q_OBJECT
public:
    Thread() {
        moveToThread(this); // WRONG
    }
 
    /* ... */
};
```

这确实可以工作（因为现在线程对象所在的线程的确改变了），但是这是个非常糟糕的设计。错误在于我们误解了 thread 对象（[QThread][] 子类）的目的：*[QThread][] 对象不是线程本身*；它是用于管理线程的，因此它应该在另一个线程中使用（通常就是创建它的线程）。

一个好的办法是：把“工作”部分从“控制”部分分离出来，创建 [QObject][] 子类对象，然后使用 `QObject::moveToThread()` 来改变对象所在的线程：

``` cpp
class Worker : public QObject
{
    Q_OBJECT
 
public slots:
    void doWork() {
        /* ... */
    }
};
 
/* ... */
QThread *thread = new QThread;
Worker *worker = new Worker;
connect(obj, SIGNAL(workReady()), worker, SLOT(doWork()));
worker->moveToThread(thread);
thread->start();
```

###应该做&不应该做

**你可以…**

- 在 [QThread][] 子类中添加信号。这是很安全的，而且可以“正确工作”（前面提到；发送者所在线程是无关紧要的）

**你不应该…**

- 使用 `moveToThread(this)`
- 强制连接类型：这通常说明你在做一些错误的事情，例如混合了 [QThread][] 控制接口和程序逻辑（它应该在该线程创建的对象中）
- 在 [QThread][] 子类中增加槽函数：它们会在“错误的”线程中被调用，不是在 [QThread][] 管理的线程中，而是在 [QThread][] 对象创建的线程，迫使你使用 direct connection 或使用 `moveToThread(this)` 函数
- 使用 `QThread::terminate` 函数

**禁止…**

- 在线程还在运行时退出程序。应使用 QThread::wait 等待线程终止
- 当 [QThread][] 管理的线程还在运行时，删除 [QThread][] 对象。如果你想要“自动析构”，你可以将 `finished()` 信号连接到 `deleteLater()` 槽函数上


##什么时候应该使用线程？
###当使用阻塞 API 时
如果你需要使用没有提供非阻塞API的库（例如信号槽，事件，回调函数，等），那么避免阻塞事件循环的唯一解决方案就是开启一个进程或线程。由于创建一个工作进程，让它完成任务并通过进程通信返回结果与开启一个线程相比是困难并且昂贵的，所以创建一个线程是更普遍的做法。

**地址解析**（只是举个例子，不是在讨论蹩脚的 API。这是每一个 C 语言函数库中包含的东西）就是一个很好的例子，它把主机名转换为地址。它会调用域名解析系统（DNS）来查询。虽然一般情况下，它会立即返回，但是远程服务器有可能故障，有可能丢包，有可能网络突然中断，等等。简而言之，它可能需要等待很长时间才相应我们发出的请求。

UNIX 系统中的标准 API 是阻塞的（不仅仅是旧的 API `gethostbyname(3)`，新的更好的 `getservbyname(3)` 和 `getaddrinfo(3)` 也是一样）。[QHostInfo][] [doc.qt.nokia.com] 是处理主机名解析的 Qt 类，它使用 [QThreadPool][] 来使得请求在后台运行（看[这里][50] [qt.gitorious.com]；如果线程支持被关闭的话，它会切换为阻塞方式）。

  [QHostInfo]: http://doc.qt.nokia.com/latest/qhostinfo.html
  [50]: http://qt.gitorious.com/qt/qt/blobs/master/src/network/kernel/qhostinfo.cpp

另一个简单的例子是图像加载和缩放。[QImageReader][] [doc.qt.nokia.com] 和 [QImage][] [doc.qt.nokia.com] 只提供阻塞方法来从设备读取图像，或改变图像的分辨率。如果你正在处理非常大的图像，这些操作可能会花费数十秒。

  [QImageReader]: http://doc.qt.nokia.com/latest/qimagereader.html
  [QImage]: http://doc.qt.nokia.com/latest/qimage.html


###当你想要充分利用多CPU时
多线程可以让你的程序更好的利用多处理器系统。每个线程是由操作系统独立调用的，如果你的程序运行在这样的机器上，线程调度就可以让多个处理器同时运行不同的线程。

比如，考虑一个批量生成缩略图的程序。一个有 n 个线程的**线程农场**（有固定线程数目的线程池），n 是系统中可用 CPU 的数量（可参考 `QThread::idealThreadCount()`），它可以将处理任务分布到多个cpu上，这样我们就可以获得与cpu数量有关的效率线性增长（简单的，我们把CPU考虑为瓶颈）。


###当你不想被阻塞时
呃…从一个例子开始会更好。

这是一个高级话题，你可以暂时忽略。Webkit 中的 QNetworkAccessManager 是一个很好的例子。Webkit 是一个流行的浏览器引擎，它是处理网页布局和显式的一组类的集合，Qt 中 QWebView 类使用了它。

QNetworkAccessManager 是 Qt 中处理 HTTP 请求和响应的类，我们可以把它当作浏览器的引擎。Qt 4.8 之前，它没有使用任何工作线程；所有的处理都在 QNetworkAccessManager 和 QNetworkReply 所在的同一个线程。

虽然在网络通信中使用线程是一个好办法，但是它也存在问题：如果你没有尽快从 socket 中读取数据，内核缓冲会被其他数据填充，数据包将被丢掉，可想而知，数据传输速率将下降。

socket 活动（也就是 socket 是否可读）是由 Qt 的事件循环还管理的。阻塞事件循环会导致传输性能下降，因为这时没有人会被告知现在数据已经可读（所以没有人会去读取数据）。

但是什么会阻塞消息循环？可悲的是：WebKit 自己阻塞了消息循环。一旦消息可读，Webkit 开始处理网页布局。不幸的是，这个处理是复杂而昂贵的，它会阻塞消息循换一（小）会儿，但足以影响传输效率（宽带连接这里起到了作用，在短短几秒内就可填满内核缓存）。

总结一下，这个过程发生的事情：

- Webkit 发起请求
- 一些响应数据开始到达
- Webkit 开始使用到达的数据来网页布局，阻塞了事件循环
- 没有了事件循环，操作系统接收到了数据，但没有人从 QNetworkAccessManager 的 socket 中读取数据
- 内核缓冲将被其他数据填充，从而导致传输效率下降

整个页面的加载时间由于 Webkit 自己引起的问题而变得很慢。

注意，由于 QNetworkAccessManager 和 QNetworkReply 都是 QObject，它们都不是线程安全的，因此你不能将它移动到另一个线程然后继续在你的线程中继续使用它，因为你可能从两个线程中同时访问它：你自己的线程和它所在的线程，因为它所在的消息循环会将事件分发给它处理。

在 Qt 4.8 中，QNetworkAccessManager 现在默认使用单独的线程处理 HTTP 请求，因此 UI 反应慢和系统缓冲被填充过快的问题得以解决。


##什么时候不应该使用线程？
###定时器
这可能是最糟糕的线程滥用。如果你不得不重复调用一个方法（例如，每秒调用一次），很多人会这么做：

``` cpp
// VERY WRONG
while (condition) {
    doWork();
    sleep(1); // this is sleep(3) from the C library
}
```

然后会发现这**阻塞了事件循环**，然后决定使用线程来解决：

``` cpp
// WRONG
class Thread : public QThread {
protected:
    void run() {
        while (condition) {
            // notice that "condition" may also need volatiness and mutex protection
            // if we modify it from other threads (!)
            doWork();
            sleep(1); // this is QThread::sleep()
        }
    }
};
```

一个**更好更简单**的办法是使用计时器，一个超时时间为1秒的 [QTimer][] [doc.qt.nokia.com] 对象，和 `doWork()` 槽函数：

  [QTimer]: http://doc.qt.nokia.com/latest/qtimer.html

``` cpp
class Worker : public QObject
{
    Q_OBJECT
 
public:
    Worker() {
        connect(&timer, SIGNAL(timeout()), this, SLOT(doWork()));
        timer.start(1000);
    }
 
private slots:
    void doWork() {
        /* ... */
    }
 
private:
    QTimer timer;
};
```

我们所需要做的就是启动一个消息循环，然后 doWork() 函数会每一秒调用一次。


###网络通信/状态机
下面是一个非常常见的网络通信的设计：

``` cpp
socket->connect(host);
socket->waitForConnected();
 
data = getData();
socket->write(data);
socket->waitForBytesWritten();
 
socket->waitForReadyRead();
socket->read(response);
 
reply = process(response);
 
socket->write(reply);
socket->waitForBytesWritten();
/* ... and so on ... */
```

不用多说，这些 waitFor*() 函数调用会阻塞消息循环，冻结 UI，等等。注意，上面的代码没有任何的错误处理，不然它会更繁琐。上面的错误在于我们忘记了最初**网络设计的就是异步的**，如果我们使用同步处理，那就是朝自己的脚开枪。解决上面的问题，许多人会简单的把它移动到不同的线程中。

另一个更抽象的例子：

``` cpp
result = process_one_thing();
 
if (result->something())
    process_this();
else
    process_that();
 
wait_for_user_input();
input = read_user_input();
process_user_input(input);
/* ... */
```

它和上面网络的例子有着同样的陷阱。

让我们退一步，从更高的视角来看看我们构建的东西，我们构建了一个状态机来处理输入。

- 空闲 –> 连接中（调用 connectToHost()）
- 连接中 –> 已连接 （发出 connected() 信号）
- 已连接 –> 发送登陆数据（发送登陆数据到服务器）
- 发送登陆数据 –> 登陆成功（服务器返回 ACK）
- 发送登陆数据 –> 登陆失败（服务器返回 NACK）

等等。

现在，我们有很多办法来构建一个状态机（Qt 就为我们提供了一个可使用的类：[QStateMachine][] [doc.qt.nokia.com]），最简单的办法就是使用枚举（整型）来记录当前的状态。我们可以重写上面的代码：

  [QStateMachine]: http://doc.qt.nokia.com/4.7/qstatemachine.html

``` cpp
class Object : public QObject
{
    Q_OBJECT
 
    enum State {
        State1, State2, State3 /* and so on */
    };
 
    State state;
 
public:
    Object() : state(State1)
    {
        connect(source, SIGNAL(ready()), this, SLOT(doWork()));
    }
 
private slots:
    void doWork() {
        switch (state) {
            case State1:
                /* ... */
                state = State2;
                break;
            case State2:
                /* ... */
                state = State3;
                break;
            /* etc. */
        }
    }
};
```

“source” 对象和“ready()”信号是什么？我们想要的是：拿网络例子来说，我们想要把 `QAbstractSocket::connected()` 和 `QIODevice::readyRead()` 连接到我们的槽函数上。当然，如果再多些槽函数更好的话，我们也可以增加更多（比如错误处理的槽函数，由 `QAbstractSocket::error()` 事件来发起）。这是真正的异步，事件驱动的设计！


###把任务分解成小块
想想一下我们有个很耗时但是无法移动到其它线程的任务（或者根本不能移动到其它线程，因为它可能必须在 UI 线程中执行）。如果我们**把任务分解成小块**，那么我们就可以返回事件循环，让事件循环分发事件，然后让它调用处理后续任务块的函数。如果我们还记得 queued connection 如何实现的话，那就很容易解决这个问题了：事件发送到接收者所在的事件循环中，当事件被分发的时候，相应的槽函数被调用。

我们可以使用 `QMetaObject::invokeMethod()` 函数，用参数 `Qt::QueuedConnection` 指定连接类型，来实现这个功能；这需要函数可调用，也就是说函数必须是个槽函数或者使用了 `Q_INVOKABLE`宏修饰。如果我们还要给函数传递参数，那么我们要保证参数类型已经通过函数 `qRegisterMetaType()` 注册到了 Qt 的类型系统中。下面的代码给我们展示了这种做法：

``` cpp
class Worker : public QObject
{
    Q_OBJECT
public slots:
    void startProcessing()
    {
        processItem(0);
    }
 
    void processItem(int index)
    {
        /* process items[index] ... */
 
        if (index < numberOfItems)
            QMetaObject::invokeMethod(this,
                                     "processItem",
                                     Qt::QueuedConnection,
                                     Q_ARG(int, index + 1));
 
    }
};
```

因为这里没有线程调用，所以它可以很容易的暂停/恢复/取消任务，也可以很容易的得到计算结果。

####参考
- Bradley T. Hughes: [You’re doing it wrong…][60] [labs.qt.nokia.com], Qt Labs blogs, 2010-06-17
- Bradley T. Hughes: [Threading without the headache][61] [labs.qt.nokia.com], Qt Labs blogs, 2006-12-04

####分类
- [Developing with Qt][62]

  [60]: http://labs.qt.nokia.com/2010/06/17/youre-doing-it-wrong/
  [61]: http://labs.qt.nokia.com/2006/12/04/threading-without-the-headache/
  [62]: http://qt-project.org/wiki/Category:Developing-with-Qt
