---
layout: post
title: "4年前的用 Django 实现的玩具 blog"
date: 2012-06-26 19:28
comments: true
categories: [python, django]
---

最近整理硬盘，发现角落里4年前写的玩具 blog，虽然现在还偶尔写写 [python][] 处理简单的事情，不过由于工作与 web 没半毛钱关系，很久都没有关注，[Django][] 什么的早就忘得差不多了。

08年9月时候 [Django][] 刚刚发布 1.0，那时候在学校时间充裕，加上关注 [Django][] 也有一段时间了，[Django][] 从 0.96 停滞这么长时间才发布 1.0，而且变化不小，于是下决心用 [Django][] 完成个小玩意儿，自己也没什么主意，不知道做什么，想来想去还是做个 blog 比较简单，到时候自己也可以用，即练手又能用自己，还能激励自己不断更新，于是花了2周多时间用 [Django][] 1.0 实现了这个玩具 blog。（当时的想法挺好，不过由于一些原因，形成雏形之后就没再更新过，所以用过一段时间就没再用了…，然后，就没有然后了…）

  [python]: http://python.org/
  [Django]: https://www.djangoproject.com/

<!-- more -->
功能实现的比较简单，不过最基本的东西都有了，文章分类，评论，按时间归档，标签，RSS，i18n(不过只有英语和汉语)，链接等等。整个过程还算顺利，有 [Django][] 的 [ORM][]  数据库的构建简单了许多，那时候 [python][] 用了有一段时间（虽然现在看起来代码很垃圾，相当的不 pythonic ），view 写起来也很顺手，但 template 就比较痛苦了，由于没有美术基础，前端也仅限于能看懂改改而已，主题 [Theme Codename H][30] 是从 [livesino][] 一个个的静态页面上扒下来的，fav 图标是借用的 [google desktop][gdesktop] 图标（很遗憾 [google desktop][gdesktop] 已于 2011年9月宣布不再提供下载，也不再为已安装的用户提供更新），logo 是直接用的[方正喵呜体][31]（现在看着还是很萌 :) ），原本想选用 [TinyMCE][] 做富文本编辑器（ [wordpress][wp] 一直使用的 [TinyMCE][]，功能很强大），但使用中遇到一些问题（现在已经想不起来是什么问题了… :( ）， 于是选择了同样老牌的 [FCKeditor][] （话说 [FCKeditor][] 的漏洞不少 –_-||，另外 [FCKeditor][] 从09年升级到3.0的时候改名为 [CKEditor][] 了），当时就想要功能强大，现在觉得无论是 [TinyMCE][] 还是 [FCKeditor][] 都太重量级了。

  [ORM]: http://en.wikipedia.org/wiki/Object-relational_mapping
  [livesino]: http://livesino.net/
  [gdesktop]: http://desktop.google.com/
  [TinyMCE]: http://www.tinymce.com/
  [wp]: http://wordpress.org/
  [FCKeditor]: http://ckeditor.com/
  [CKEditor]: http://ckeditor.com/
  [30]: http://livesino.net/theme-codename-h#mobile
  [31]: http://www.foundertype.com/showsortpic.php?inputword=%B0%AE%D3%DA%D7%D6%C0%EF%D0%D0%BC%E4&sid=1041&fontfile=FZMWFONT.TTF

最近闲来无事，把这个玩具迁移到 [Django] 1.4了，从 1.0 到 1.4 变化不少，[The syndication feed framework][40] 在 1.2 版本的时候重构了，1.2 之前的接口在 1.4 版本中移除了，不过原本 feed 的功能就使用的不多，移植起来比较容易，另外 1.4 默认开启了中间件 [CSRF][]，使用起来也很容易，还有一些其他的小改动就不一一列举了，最大的改动应该是 1.3 引入的 [staticfiles app][41]，这使得部署静态资源变得更容易了，在引入 staticfiles 之后，只要执行 manage.py collectstatic 就可以方便的将app中所用到的静态资源拷贝到同一目录了。

用了几天的空闲时间把这个简陋的 blog 的迁移任务初步完成了，目前自己没有主机，也没有时间打理，关于如何部署到服务器可以参考 [Deploying Django][50]，数据库可以参考 [Databases][]，如果还有空闲的话希望能够完善，虽然已经4年没有做过web项目了，但希望如果有机会，以后还能参与。不说废话了，上两张图吧。

  [50]: https://docs.djangoproject.com/en/1.4/howto/deployment/
  [Databases]: https://docs.djangoproject.com/en/1.4/ref/databases/

![](https://raw.github.com/bitdewy/MONShellog/master/snapshot00.png)
![](https://raw.github.com/bitdewy/MONShellog/master/snapshot01.png)

最后，项目地址：[MONShellog](https://github.com/bitdewy/MONShellog)
