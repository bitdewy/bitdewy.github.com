---
layout: post
title: "Unix 的隐藏文件是个 Bug"
date: 2012-09-14 01:05
comments: true
categories: [unix]
---

玩过 Unix/Linux 的人应该都知道，在 *nix 文件系统中，以`.`开始的文件(夹)是隐藏文件(夹)，但你可能想不到这是一个设计上的 bug。

8月份的时候，[Rob Pike][Rob] 在 Google+ 上发了一篇名为 "[A lession in shortcuts][00]" 的短文，介绍了 Unix 文件系统中隐藏文件这个设计上的 bug (feature)。

  [Rob]: http://en.wikipedia.org/wiki/Rob_Pike
  [00]: https://plus.google.com/101960720994009339267/posts/R58WgWwN9jp

在 Unix 文件系统设计的早期，为了导航更方便而引入了 `.` 和 `..`。当用户输入 `ls` 时会列出文件清单，所以 [Ken][] 或 [Dennis][] 添加了一些简单的代码。当时用的是汇编，不过代码看起来应该是像这样的：

  [Ken]: http://en.wikipedia.org/wiki/Ken_Thompson
  [Dennis]: http://en.wikipedia.org/wiki/Dennis_Ritchie
``` c
if (name[0] == '.')
    continue;
```

这比它应有的样子稍微短了一点点：

``` c
if (strcmp(name, ".") || strcmp(name, "..") == 0)
    continue;
```

嘿，这很简单。

<!-- more -->
但产生了两个后果。

首先，这不是一个好的先例。一些其他懒惰的程序员用相同的简化方式引人了 bug。本应该被计算的以 `.` 开始的文件就这样被忽略掉了。

其次，更糟糕的是，隐藏文件的概念随之而来了。更懒惰的程序员开始往 home 目录中仍各种文件(夹)。在我写这片短文的机器上，我没有安装多少东西，但是 home 文件夹下有数以百计的隐藏文件，我甚至不知道大部分是干什么的，也不知道它们是否还有用。每次需要扫瞄 home 目录的操作都会因为这些长时间积累下来的垃圾而变慢。

我非常确定，隐藏文件的概念是一个无意中产生的结果。这完全就是一个错误。

原文地址：[A lession in shortcuts][00]

PS： 上面一共出现了三个人，这三位都是神级的人物，网络上有一篇流传了很多年的文章叫做 “我心目中的编程高手” 里面介绍了 9 位神级人物，这三位大神都榜上有名。


