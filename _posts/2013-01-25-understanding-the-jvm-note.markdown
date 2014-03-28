---
author: superpippo
comments: true
date: 2013-01-25 15:03:50+00:00
layout: post
slug: understanding-the-jvm-note
title: 《深入理解Java虚拟机》读书笔记
wordpress_id: 463
categories:
- programming
- reading
tags:
- java
- jvm
- 阅读
---

这本书是13年看的第一本技术书，算是采用了分析阅读的方法来阅读。尝试着以问自己四个问题的方式（[来源](http://book.douban.com/subject/1013208/)），来做一次读书笔记。

## 这本书主要写了什么？

围绕着JVM这个技术体系，重点阐述了JVM如何对内存进行分配，垃圾的回收方式（GC），字节码的结构与生成机制，类加载器的工作机制，JVM解释执行与编译执行（JIT）的细节，最后提到了Java内存类型和Java线程的设计原则。需要注意的是：JVM是一种规范，或者说是概念模型，而各虚拟机厂商在实现上会有一些区别，在阅读时必须注意JVM这个词的上下文语境。

## 这本书是怎么组织的？

### 1) 走近Java

这一章其实是不太好写的，无非是大量的摘录和罗列，并不容易出彩。      
内容上简要介绍了Java技术体系，JDK的发展历史，未来方向的展望，最后编译openjdk的实战例子奠定了全书「干货」颇多的基调。

### 2) 自动内存管理机制

这一部分提出并解释了这么一系列问题：      
什么操作可能导致内存溢出？有哪些种类的内存溢出？都是在内存的哪些区域溢出的？       
一般说来，垃圾收集有哪些原则？JVM有哪些垃圾收集算法及实现？新生代和老年代的回收策略如何？各种内存相关的JVM参数都是什么意思？

### 3) 虚拟机执行子系统

首先从数据结构的粒度解释字节码（class文件）的每一部分细节，这一部分看的相当过瘾。      
然后阐述了JVM在类加载的各个阶段都做了哪些事情，并简要介绍了JVM类加载器与OSGi类加载器的异同。       
最后从概念模型的角度（即解释执行）说明了虚拟机执行引擎是如何执行字节码的。       
PS：这一部分比较多的联系到JVM内存结构（第2章）和字节码文件结构（第6章）的知识，需要多看几遍，将第二部分和第三部分吃透。

### 4) 程序编译与代码优化

这一部分首先介绍了虚拟机如何将程序源代码编译成字节码（即Javac编译器实现），书上称之为「前端编译」。      
PS：这一部分让人回忆起遥远的编译原理，当时还是[龙书](http://book.douban.com/subject/1134994/)的译者姜守旭老师给我们上的课呢，哈哈。。       
然后阐述了在字节码执行时虚拟机的具体实现（承接第8章的内容，这里主要介绍了编译执行即JIT的优化执行过程）。

### 5) 高效并发

这一部分简要介绍了Java内存模型（JMM），Java线程的实现，与操作系统线程之间有什么联系，最后介绍了Java中线程安全的语义（并不是非黑即白的二元语义）和锁优化。

## 这本书说的都对吗？

这一点其实通过读书时感受作者踏实严谨的态度，以及作者在javaeye社区中的口碑，已经不需要太怀疑。但并不代表你不需要以实践的态度去敲书中的代码，或是编程序来验证自己的理解，这跟质疑作者的权威无关。      
例如在介绍[volatile](https://www.ibm.com/developerworks/java/library/j-jtp06197/)关键字的时候，我手写了一个例子去验证和理解「volatile关键字会保证当一条线程修改了这个变量的值，新值对于其他线程来说是立即得知的」这句话。但比较遗憾，例子并不work。SO上也有很多人在讨论这个问题：[link1](http://stackoverflow.com/questions/5816790/the-code-example-which-can-prove-volatile-declare-should-be-used)，[link2](http://invalidcodeexception.com/java-volatile-keyword-by-example/)。所以对于虚拟机这个大背景，需要分清什么是「规范」，而什么是各类虚拟机的实际「实现」，这里面会有很多东西值得深究。

## 这本书对我有什么意义？

其实在买这本书前我也在想这个问题，毕竟平时的开发工作和JVM相去甚远。但这其实是种很功利的想法，读这本书其实是修炼内功，提高自身竞争力的一种手段，可能一时半会并不会有太大的收益，甚至需要你要在之后持续且不浮躁地投入相当的精力去钻研相关的技术，但这一切都是值得的。

下面结合我自身情况，列出了以后学习的一些方向：

1. 若要钻研JVM，那[Java虚拟机规范文档](http://icyfenix.iteye.com/blog/1256329)是必读的，还有莫枢的这篇[中文文档](http://rednaxelafx.iteye.com/blog/858009)也是必读的。       
1. 理解[JMM](http://www.iteye.com/topic/1128781)能更深入理解Java中的线程，并发（[link1](http://book.douban.com/subject/1888733/)，[link2](http://book.douban.com/subject/20142617/)）。       
1. 从Javac编译器实现的角度理解各种语法糖，如[泛型](http://www.ibm.com/developerworks/java/library/j-jtp01255/index.html)与类型擦除，自动装箱拆箱等。这样读[Effective Java](http://book.douban.com/subject/3360807/)和[Java解惑](http://book.douban.com/subject/5362860/)的例子时，说不定javap一下会有恍然大悟的感觉。       
1. 尝试着编写程序「量化」地分析产品的类加载时间，分析java core/dump和性能瓶颈（有时候不能过于依赖于各种GUI）。       
1. 深入了解[OSGi](http://book.douban.com/subject/3574458/)的类加载。（据说作者年后要出一本《深入理解OSGi》）
