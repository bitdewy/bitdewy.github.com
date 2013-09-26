---
layout: post
title: "模板方法模式"
date: 2013-05-28 01:26
comments: true
categories: [OO, design pattern] 
---

##意图

1. 定义一个操作中的算法骨架, 将一些步骤延迟到子类中定义. 模板方法模式, 可以让子类不改变算法的结构重新定义算法的某些步骤.
2. 基类声明算法的“占位符”, 子类来实现这个“占位符”.
 
##讨论

组件设计者决定哪些算法的步骤是不变的? 哪些是会变化的? 不变的步骤在抽象类中直接实现, 可能变化的步骤给出默认的实现, 或者完全不实现. 变化的部分代表“hooks”, 或占位符. 可以在使用组件时, 在派生类中实现.

组件设计者设计算法的步骤, 步骤的顺序, 但同时允许用户来扩展或替换算法中的某些步骤.

模板方法模式在框架中的使用非常广泛. 每个框架都会实现框架的主要结构, 然后定义一系列的“占位符”给用户, 让用户有机会来自定义一些操作.这么做时, 框架就变成了核心部分, 用户自定义的操作就会变得容易. 这使得控制结构发生反转, 这个方式有一个很好的名字, “好莱坞原则”——“不要给我们打电话, 我们会打电话给你.”

<!-- more  -->
##结构

![](https://qhphgq.bay.livefilestore.com/y2pBx_PXPtSilwtV5sYskGDPJkXjmpbdLnwAeT9H_7drKPs4-4_-pm3OfDw58eFEjaFRaa2LSKhbnJODUkyO8CpuQRq8iF306c3KF6Ix_M1zWA/image002.gif)
 
template_method() 的实现如下: 调用 step_one(), 调用 step_two(), 然后调用 step_three(). step_two() 是一个 “hook” 方法——一个占位符. 它在基类中声明, 然后在派生类中实现. 框架中使用模板方法模式非常的频繁. 所有可重用的代码都定义在框架的基类中, 框架的用户可以通过派生类来定制一些想要的行为.

![](https://qhphgq.bay.livefilestore.com/y2pMCc56J8x2oBUDmOTnSI3RIm-N5Gtpyj355-us-8Rj6CwVbNzQfiEJyprTSAXmCS8rG0MbCE-qwP-Y65dPUdVWyXU8V2FC4IWhzy_2WBvDZA/image003.gif)
 
##做法

1. 检测算法, 决定哪些步骤是标准的通用的, 哪些是特有的.
2. 定义一个抽象类, 用来实现依赖的反转.
3. 将算法的壳以及所有的标准（公共）的步骤移到基类中.
4. 在基类中给那些需要不同实现的步骤定义一个占位符或者“hook”方法, 这些方法可以有默认实现, 也可以完全没有. (C++中的纯虚函数, Java中的抽象方法）
5. 在模板方法中调用这些“占位符”.
6. 所有现有的类都继承于现在的抽象类.
7. 将基类中已经实现的现有的类中的细节全部移除.
8. 那些不同的部分仍然留在这些派生类中，用来实现每个派生类中不同的操作.
 
##经验

除了粒度的区别之外, 策略模式与模板方法模式非常类似.
模板方法使用继承来改变算法的某个部分, 而策略模式改变整个算法.
策略模式改变单个对象的逻辑. 模板方法修改了整个类的逻辑.
工厂方法模式是一个特化的模板方法模式.
 
##例子

```cpp 
#include <iostream>
using namespace std;
class Base {
    void a() { cout << "a  "; }
    void c() { cout << "c  "; }
    void e() { cout << "e  "; }
 
    // 2. Steps requiring peculiar implementations are "placeholders" in base class
    virtual void ph1() = 0;
    virtual void ph2() = 0;
 
public:
    // 1. Standardize the skeleton of an algorithm in a base class "template method"
    void execute() {
        a();
        ph1();
        c();
        ph2();
        e();
    }
};
 
class One: public Base {
    // 3. Derived classes implement placeholder methods
    /*virtual*/void ph1() { cout << "b  "; }
    /*virtual*/void ph2() { cout << "d  "; }
};
 
class Two: public Base {
    /*virtual*/void ph1() { cout << "2  "; }
    /*virtual*/void ph2() { cout << "4  "; }
};
 
int main() {
    Base* array[] = { &One(), &Two() };
    for (int i = 0; i < 2; i++) {
        array[i]->execute();
        cout << '\n';
    }
    return 0;
}
```
