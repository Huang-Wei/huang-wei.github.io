---
title: Blog继续搬家
tags: blog jekyll github
lang: zh
---

从今天起，位于[lifebackup.cn](http://www.lifebackup.cn)的blog写作将全迁移至此，之后的某个时间域名也将直接指向此。Bye, wordpress!

<!--more-->

## Blogdriver and MSN Space（2005.1-2007.12）

我是大二的时候有一搭没一搭的开始写博客的，那个时代也是各博客提供商争夺用户最激烈的时代。受俱乐部师兄影响，在Blogdriver动力上开了一个博客（确实得用“开”这个词，因为真的真的是毫无技术含量可言）。那个时候的博客无非是提供几个模板，博客写作大致也就是在word里写好，贴进后台编辑器，倒饬倒饬段落样式，齐活，提交。

这时候的blog你没有定制，没有自由，没有插件，你只能接受提供商给你的东西。在Blogdriver之后，图新鲜换到了MSN Space，但写作方式和blog所能提供给你的内容几无二致。

## Wordpress (2008.1-2013.11)

这是一段写blog能给人带来乐趣的时光，自己注册域名，用cowoo提供的免费空间写作。在wordpress里，每一行blog代码，每一个插件都能由你选择，修改，这种你有100%权力控制一切的感觉非常棒。这段时间我一直用windows作为我的开发系统，所以我选用了windows live writer作为写作工具。

也曾鞭策自己一个月至少要写一篇，现在回过头来看这个wordpress blog的[诞生](http://www.lifebackup.cn/my-blog-history.html)，还能感觉到那时的那份激情：）

wordpress后台的截图：
![](/images/201312/wordpress.png)

## Jekyll hosted on Github (2013.12)

说实话wordpress是非常棒的博客，简单易学，生态系统也极其完善。但我决定放弃wordpress的原因有两点：

1. 从8月份起，我的开发机完全转到ubuntu，so no more mircrosoft stuff... 之后我一直在找一个linux下好用的blog编辑器，但没有live writer的远程提交功能，发现都还不如放到wordpress后台直接写。简洁的markdown是我的首选，但考查的几个markdown for wordpress的插件均不理想。

1. 众所周知，wordpress的blog是放在mysql里的，这使得修改，备份文章都需要花费额外的工作。但Jekyll是一种静态的博客系统，它并不使用数据库，直接把文件系统符合一定格式的（源）文章，通过markdown引擎编译。对于作者来说，修改和保存源文章就是你所需要花费的全部工作，这非常方便。而由于Github pages后台就是使用的Jekyll，那么把文章放在github，本地修改，通过git提交，这就是blog的一个完整流程了。这个blog生态系统对于程序员来说，实在是找不出瑕疵。

所以我现在的写作流程就是：用sublime以markdown的语法写作（之前可能先把想法丢到_drafts里），本地`jekyll serve --watch`随时看文章的效果。当决定发表时，提交到github。

## 后记
阮一峰说一个人的blog大概会经过[三个阶段](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html):

> 第一阶段，刚接触Blog，觉得很新鲜，试着选择一个免费空间来写。
>
> 第二阶段，发现免费空间限制太多，就自己购买域名和空间，搭建独立博客。
>
> 第三阶段，觉得独立博客的管理太麻烦，最好在保留控制权的前提下，让别人来管，自己只负责写文章。

而我的个人blog也莫出其外，趁还记得，把这些逝去的blog历史写下，权当记念。