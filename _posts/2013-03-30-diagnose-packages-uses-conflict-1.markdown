---
author: superpippo
comments: true
date: 2013-03-30 15:19:09+00:00
layout: post
slug: diagnose-packages-uses-conflict-1
title: 如何诊断OSGi环境下的packages uses conflict（一）
wordpress_id: 497
categories: programming
tags:
- java
- osgi
---

[之前的文章](http://www.lifebackup.cn/osgi-bundle-dependency.html)提到了在OSGi环境下会出现Packages uses conflict问题，在那个例子中，产生的原因在于：

1. 上游的bundle(B)在export package(u.v.w.common)时使用了`:uses`关键字，进而严格绑定(wire)了某个特定package(x.y.z.core 1.0.0)  
1. 当前的OSGi环境下有多个同名的export package(x.y.z.core 1.0.0和2.0.0)  
1. 下游的bundle(C)直接或间接地依赖多个同名的package(x.y.core 1.0.0是间接依赖，x.y.z.core 2.0.0是直接依赖)。而如果不是import package，而是require bundle，情况会更加简单粗暴——因为require bundle等价于硬性import那个bundle的所有export package。

了解这三个原因，对诊断OSGi中的此类问题非常重要。

其中#1的产生，有时候是无能为力，因为这是spring等一些包的常用做法。所以既然你选择了spring，那它就需要了解它的export package是会带有大量:uses的。

\#2的产生有时候是这样的：例如在我所开发的产品中，分为client和server两部分。有些bundle是共用的，其export了常用的package。但是client端是基于Eclipse的，而Eclipse会默认带入大量的package，这就有可能与常用的那部分package有重复的地方。

\#3的产生说到底，还是因为spring内部大量使用了:uese，而且均使用Import Package的方式设置依赖。这在带来灵活性的同时，也注定了在运行期前很多行为是不可以预知的——例如spring究竟是import了哪个bundle的package？

下面看两段OSGi命令行的输出，它们产生于两次真实的运行时环境：

```
case #1  
osgi> p org.apache.commons.collections  
org.apache.commons.collections; version="0.0.0"<eclipse.provider [106]>  
org.springframework.bundle.spring.context.support_2.5.5 [1513] imports  
org.springframework.bundle.spring.core_2.5.5 [1514] imports  
......  
org.apache.commons.collections; version="0.0.0"<shared.provider [578]>  
......
```

```
case #2  
osgi> p org.apache.commons.collections  
org.apache.commons.collections; version="0.0.0"<eclipse.provider [104]>  
......  
org.apache.commons.collections; version="0.0.0"<shared.provider [575]>  
org.springframework.bundle.spring.context.support_2.5.5 [1511] imports  
org.springframework.bundle.spring.core_2.5.5 [1512] imports  
......
```

看出蹊跷来了吧：这两个spring bundle在两次运行期import的是不同bundle提供的org.apache.commons.collections。这好像没什么大不了是吗？那么假设test.bundle同时require org.springframework.bundle.spring.core_2.5.5和shared.provider，会发生什么事情呢？

![image](/images/201303/image.png)

![image](/images/201303/image1.png)

是的，case #1中的test.bundle不会被正确加载，它的状态仅仅是installed，诊断信息显示为：

```
reference:file:/D:/.../test.bundle.jar [437]  
Package uses conflict: Require-Bundle: org.springframework.bundle.spring.context.support; bundle-version="0.0.0"  
Package uses conflict: Require-Bundle: org.springframework.bundle.spring.core; bundle-version="0.0.0"
```

OK，到目前为止，我们以反向角度一步步地解释并验证了packages uses conflict产生的原因。[下一篇](http://www.lifebackup.cn/diagnose-packages-uses-conflict-2.html)文章将介绍遇到这种问题时，如何正向的找到最根的那个有问题的bundle（test.bundle），以及哪个package才是真正冲突（org.apache.commons.collections），最后给出可能的解决方案。

  

