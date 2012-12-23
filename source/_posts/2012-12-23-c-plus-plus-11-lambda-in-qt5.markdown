---
layout: post
title: "C++11 lambda in Qt5"
date: 2012-12-23 22:32
comments: true
categories: [C++, Qt]
---

C++11 发布已经1年多，各家编译器也都有了很好的支持，新特性中有非常多值得关注的东西，比如：右值引用、类型推导、匿名函数等等。

##lambda 函数与表达式

C++11 引入了 lambda 函数，在函数是 first class 的语言中，匿名函数是最基础的设施，可以方便的运用闭包造出高阶函数。现在 C++11 引入了 lambda 表达式（是不是可以在 C++ 中玩 Functional Programming 了，嘿嘿)，和标准库算法配合起来就不会像以前那么别扭了。在没有 lambda 的时候使用 `std::sort` 或者 `std::find` 时，需要一个具名函数，原本一个很简单的，用完就丢掉的代码段必须声明为一个函数或仿函数，割裂了逻辑不说，还占用了一个标识符(取名，在写代码的时候时很让人头疼的一件事，你懂的)，现在一切都变的简单了。

<!-- more -->
假设你想计算某个字符串包含多少个大写字母，使用 for_each() 遍历一个 char 数组，下面的 lambda 表达式确定每个字母是否是大写字母，每当它发现一个大写字母，lambda 表达式给 Uppercase 加 1：

``` cpp
int main() 
{
    char s[] = "Hello World!";
    int Uppercase = 0; //modified by the lambda  
    for_each(s, s+sizeof(s), [&Uppercase] (char c) {
        if (isupper(c))
            Uppercase++;
    });
 cout << Uppercase << " uppercase letters in: " << s << endl;
} 
```

匿名函数的威力当然不是只有这么一点，不过这不是本文的重点，关于匿名函数和闭包可以向会 javascript 的同学学习。


##Qt5

### old signal & slot

Qt5 终于在末日前一天——12月20日正式发布了，信号槽有了新的用法。

在 Qt5 之前，信号槽连接要这么写：

``` cpp
connect(sender, SIGNAL(valueChanged(QString,QString)),
             receiver, SLOT(updateValue(QString)));
```

Qt 利用 SIGNAL 和 SLOT 这两个宏，把函数名转换成一个字符串。然后后，moc 将会扫描全部文件，将所有的 signal 和 slot 提取出来做成一个映射表。QObject::connect() 函数会从这个映射表里面找到该字符串，从 signal 的名字就可以找到 slot 的名字，就知道了在 signal emit 的时候，该去调用哪一个 slot 函数。

但是，由于信号槽都被处理成了字符串，所以编译期间是无法检查的，所有检查都是在运行时完成的。而且，由于是字符串，所以 slot 中的参数类型名必须与 signal 的完全一致。


### new signal & slot

在 Qt5 中我们有了更好的选择（原来的语法仍然支持）：使用函数指针

``` cpp
connect(sender, &Sender::valueChanged, 
        receiver, &Receiver::updateValue);
```

看起来和之前的版本很类似，但新的用法有很多好处：

- 编译期类型检查，以及 `Q_OBJECT` 宏检查
- 可以随意使用 typedef 和 namespace 了
- 支持参数类型的隐式转换，比如(QString 转换为 QVariant)
- 可以连接任何成员函数，不仅仅是槽函数

坏处是，槽函数从此以后不能用默认参数了。(也没那么坏，是吧)

更有威力的是，connect 可以连接简单函数了，不仅仅是 QObject 对象的成员函数：

``` cpp
connect(sender, &Sender::valueChanged, someFunction);
```

看到这儿，就能想到最开始的 lambda 了吧，没错，`someFunction` 可以是个 function 也可以是一个 lambda 表达式，新的连接方式可以和 `tr1::bind(std::bind)` 以及 lambda 表达式完美的配合。

``` cpp
connect(sender, &Sender::valueChanged,
    tr1::bind(receiver, &Receiver::updateValue, "senderValue", tr1::placeholder::_1));
 
connect(sender, &Sender::valueChanged, [=](const QString &newValue) {
        receiver->updateValue("senderValue", newValue);
    });
```

唯一的坏处就是： `receiver` 在销毁的时候，无法自动断开连接


###异步操作变的更简单

有了 C++11 的 lambda 表达式，与 STL 的算法一样，关于网络的一些异步操作同样变的非常简单了。

``` cpp
void doYourStuff(const QByteArray &page)
{
    QTcpSocket *socket = new QTcpSocket;
    socket->connectToHost("qt.nokia.com", 80);
    QObject::connect(socket, &QTcpSocket::connected, [socket, page] () {
        socket->write(QByteArray("GET " + page + "\r\n"));
    });
    QObject::connect(socket, &QTcpSocket::readyRead, [socket] () {
        qDebug()<< "GOT DATA "<< socket->readAll();
    });
}
```

下面是一个 `QDialog` 的例子，代码没有被割裂，也没有重入消息循环，看起来干净多了。

``` cpp
void Doc::saveDocument() {
    QFileDialog *dlg = new QFileDialog();
    dlg->open();
    QObject::connect(dlg, &QDialog::finished, [dlg, this](int result) {
        if (result) {
            QFile file(dlg->selectedFiles().first());
            // ...
        }
        dlg->deleteLater();
    });
}
```

虽然 C++ 不推崇函数式，但 lambda 的引入绝对是一大进步，据说 Java 8 也会引入 lambda，不过 Java 8 什么时候发布都还没谱呢……
