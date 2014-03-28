---
author: superpippo
comments: true
date: 2013-02-24 04:08:24+00:00
layout: post
slug: emf-intro-ecore-model
title: EMF简介——构建ecore model
wordpress_id: 175
categories: programming
tags:
- ecore
- EMF
---

元模型作为EMF框架中的模型表示，在产品架构中起着举足轻重的作用，这篇文章将对EMF元模型构建中的一些基本概念及细节作尽可能细致的阐述。

EMF中的元模型是一个.ecore文件，你可以通过Eclipse自带的EMF模块来设计（下载Eclipse带EMF模块的build）。同时Eclipse也支持其他方式的导入，从而生成ecore：

> * 其他工具的模型文件，例如Rational Rose的UML模型，RSA中的emx模型   
> * xsd文件

本篇文章主要通过Eclipse自带的方式介绍Ecore的构建。

1. 在Eclipse中New.. –> Project –> Empty EMF Project   
2. 这样会有一个plugin生成，并自动生成一个model文件夹（默认存放模型相关的文件）   
3. 在model中New.. –> Other.. –> Ecore Model，取名BookStore.ecore

这样一个空的ecore就建好了：

![image](http://www.lifebackup.cn/wp-content/uploads/2011/09/image-thumb.png)

在继续构建ecore前，先介绍几个重要的概念：

* EPackage: 跟java的package类似，用以组织语义上相似的元素。EPackage可以有0个或多个esubPackage，但root package只有一个，并与EFactory互相关联。每个EPackage由其URI属性唯一标识，并在持久化时表示成XML根结点的namespace（nsURI）。  
在EMF内部维护了一个ePackage的hashMap可以通过nsURI得到相对应的ePackage：EPackage.Registry.INSTANCE.getEPackage(pkgUri)  
![image](http://www.lifebackup.cn/wp-content/uploads/2013/02/image.png) ![image](http://www.lifebackup.cn/wp-content/uploads/2013/02/image1.png)   
* EFactory: 单例，包含了创建此模型中各对象的create方法。   
* EClass: 相当于java的Class，由各eStructuralFeature组合而成。   
* EReference: EStructuralFeature的一种，一般为指向其他EClass（可以认为是复杂对象）。   
* EAttribute: EStructuralFeature的一种，一般为简单对象。   
* EDataType: 初次接触EMF的话会比较难懂，可从从以下几个角度几个方面理解：
  1. 一般用来代表Java中标准的对象类型，例如你需要使用Java中的IP地址类java.net.InetAddress，就可以定义一个叫InetAddress的EDataType。   
  1. 由EAttribute在EType属性中设置。（EReference中设置的EType是EClass）  
  1. EDataType所实例化的对象只会存在于内存中，而其序列化和反序列化则由一组可逆的序列规则控制。也就是说你需要_**手工**_实现从java.net.InetAddress到String的映射逻辑（createXXFromString(), converXXToString()），EMF框架并不知道这些。   
  1. EEnum可以看成特殊的EDataType，EInt, EString等可以看成EMF_**内置**_的EDataType。

![image](http://www.lifebackup.cn/wp-content/uploads/2011/09/image-thumb1.png)

了解这些EXXX的概念和设计原则，才能更加游刃有余地使用并定制化基于EMF的各式应用。（PS：这些概念对理解Dynamic EMF非常关键）

在本文中，以书店中对图书的简单建模为例（未使用到EDataType）：

[![image](http://www.lifebackup.cn/wp-content/uploads/2011/09/image-thumb2.png)](http://www.lifebackup.cn/wp-content/uploads/2011/09/image2.png)

其中EAttribute与EReference均有lowerBound(默认0)和upperBound(默认1)属性，除了0，1外，-1代表unBound，-2代表unspecified。

在ecore生成之后，就可以New.. –> Other.. –> EMF Generator Model生成一个genmodel（要以.genmodel结尾）。生成之后，在package级别上设置一些Base package和Prefix等属性，就可以自动生成（Generate Model Code）业务模型的脚手架代码。这些模型代码基于比较典型的设计模式设计，蕴含了EMF的三大特性：持久化，通知机制以及反射。不光可以用和Eclipse相关的项目中，也可以用在脱离Eclipse容器的普通Java应用中。<strike>下一篇文章将以Java Standalone Application的形式介绍如何使用这些业务模型的脚手架代码。</strike>本篇文章的示例代码[下载](http://www.lifebackup.cn/wp-content/uploads/2013/02/ecoretest.zip)。

总的来说，整个EMF模型及后续应用程度的开发是一个[MDA](http://en.wikipedia.org/wiki/MDA)的过程。首先核心是建立这个元模型（通过Eclipse源生构建的ecore，或其他工具创建的UML模型），然后利用EMF框架生成Model的脚手架代码，为之后应用程序打下基础。当然，若基于EMF的模型生成代码，就会很自然用EMF的方式（XML文件）来持久化模型。而如果想基于数据库来存储和开发应用，亦未尝不可——利用ecore模型（或是UML模型）基于Hiberbate或OpenJPA也能生成对应于数据库CRUD操作的脚手架代码。PS：有一些[项目](https://github.com/BryanHunt/mongo-emf)尝试着将EMF store持久化到MongoDB，有兴趣的读者可以尝试一下。
