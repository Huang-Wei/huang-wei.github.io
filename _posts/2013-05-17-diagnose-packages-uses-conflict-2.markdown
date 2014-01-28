---
author: superpippo
comments: true
date: 2013-05-17 13:18:28+00:00
layout: post
slug: diagnose-packages-uses-conflict-2
title: 如何诊断OSGi环境下的packages uses conflict（二）
wordpress_id: 504
categories: programming
tags:
- java
- osgi
---

[上一篇](/programming/2013/03/30/diagnose-packages-uses-conflict-1.html)文章说到packages uses conflict出现的原因，这篇文章来说一下怎么快速定位到不能加载的bundle，并找到解决方案。

## 怎么样才算有问题

从产品的功能角度来说，这类问题只会在特定的运行期显现，编译期仅仅是「潜浮」状态。而从开发的角度看，问题的出现是有很多症状的：在OSGi的控制台ss后看到一些INSTALLED的bundle始终没法start，或启动log中有大量以下的信息，都是出现问题的征兆。

```
!ENTRY org.eclipse.osgi 2 0 2013-03-30 20:26:32.242  
!MESSAGE The following is a complete list of bundles which are not resolved, see the prior log entry for the root cause if it exists:  
!SUBENTRY 1 org.eclipse.osgi 2 0 2013-03-30 20:26:32.242  
!MESSAGE Bundle bunle_x [6] was not resolved.  
!SUBENTRY 2 bunle_x 2 0 2013-03-30 20:26:32.242  
!MESSAGE Missing required bundle bunlde_y.  
!SUBENTRY 2 bunle_x 2 0 2013-03-30 20:26:32.242  
!MESSAGE Missing imported package package_z.
```

## 定位问题

这里说的就是快速定位到上个例子中的test.bundle（bundle依赖链中最顶层的那个）。  
首先是浏览一下所有INSTALLED的bundle：`ss | grep INSTALLED`  
然后根据你对代码的理解，尽量找一个在bundle依赖链中较顶层的开始入手：`diag <bundle_x #>`

```
osgi> diag 150  
reference:file:/D:/.../bundle_x.jar [150]  
Package uses conflict: Require-Bundle: bundle_y; bundle-version="0.0.0"  
Direct constraints which are unresolved:  
Missing required bundle bundle_y_0.0.0.
```

进而发现它的直接上层依赖没有正常启动，继续往上层诊断：`diag <bundle_y #>`。这样很快就会找到一个根（可能不止一个），其所有上层依赖都正常启动，但它本身是unresolved的，即test.bundle。  

## 找出真正的问题所在

找到test.bundle事情其实才刚刚开始，问题的核心在于哪些package才是真正冲突所在。  
简单的笨方法就是靠对组件间依赖的理解，修改MANIFEST.MF，不断「试错」，最终解决这个问题。  
而一劳永逸的方法就是导入eclipse中osgi的源代码了进行debug，其中几个切入的点在： 


* ResolverImpl#findBestCombination  
注意bundleConstraints, packageConstraints中的内容

* GroupingChecker#isConsistentInternal(ResolverBundle requiringBundle, ResolverBundle matchingBundle, List<ResolverBundle> visited, boolean dynamicImport, List<PackageRoots[]> results) 

* 然后可以在org.eclipse.osgi.internal.module/GroupingChecker$PackageRoots的两个isConsistentClassSpace()方法设置断点
