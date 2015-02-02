---
author: superpippo
comments: true
date: 2013-01-03 14:57:34+00:00
layout: post
slug: compile-openjdk7-ubuntu-1210
title: Ubuntu12.10下编译openjdk 7
wordpress_id: 432
categories: programming
tags:
- java
- JDK编译
- ubuntu
---

最近在读《[深入理解Java虚拟机](http://book.douban.com/subject/6522893/)》这本书，于是谨遵书中教诲，尝试着自己动手编译一个jdk。这篇文章就把在Ubuntu12.10下编译openjdk 7的*最简*步骤记录下来，以备不时之需。

## 1. 下载build所需lib包

```
sudo aptitude build-dep openjdk-7
```
 

## 2. 下载openjdk7源码

我编译时用的是[官方](http://openjdk.java.net/)打包好的源码包，这里要注意的是：**选择最新的源码包以避免不必要的麻烦**（即主页上**带7u的链接**，而不是7）。

![image](/images/201301/image.png)

## 3. 设置当前shell的环境变量(optional)

```
export LANG=C ALT_BOOTDIR=/usr/lib/jvm/java-7-openjdk-amd64 
```

这两个参数在当前shell设置是为了之后编译出错时make clean能直接生效。

## <a name="step4"></a>4. 检查编译环境的正确性

进入源码包的openjdk目录： `make sanity > ~/sanity-test.txt`。

这份输出包含了很丰富的编译配置信息，其中也可以看出一些必须的参数并未设置（搜索NOT-SET），如在Build Tool Settings中：

```
ANT_VER = Unable to locate tools.jar. Expected to find it in /usr/lib/jvm/java-7-openjdk-amd641.8.2 [requires at least 1.7.1] 
```

所以我们在执行build前需要把这些参数都设置好，下面是一份适用于Ubuntu12.10的编译脚本：

```bash
export LANG=C
export ANT_HOME=/usr/share/ant
export ALT_BOOTDIR=/usr/lib/jvm/java-7-openjdk-amd64
export DEBUG_NAME=debug

export ALT_JDK_IMPORT_PATH=/usr/lib/jvm/java-7-openjdk-amd64

make ARCH_DATA_MODEL=64 BUILD_JAXWS=false BUILD_JAXP=false
```

注意：如果你的机器事先没有装JDK，那么需要装JDK（第一步的命令仅仅装了JRE）ANT才能起作用：

```
sudo aptitude install openjdk-7-jdk
```
 
## 5. 编译

执行[第4步](#step4)的编译脚本。如果编译失败用make clean来清除残留文件。

## 6. 编译成功

```
#-- Build times ----------        
Target debug_build         
Start 2013-01-03 19:32:25         
End 2013-01-03 19:45:04         
00:01:10 corba         
00:04:57 hotspot         
00:06:03 jdk         
00:00:29 langtools         
00:12:39 TOTAL         
-------------------------         
make[1]: Leaving directory '/home/hweicdl/dev/openjdk'
```

## 7. 检查结果

成功后在build目录下，有一个编译好的jdk出现。执行一下`java –version`看效果吧~!

![image](/images/201301/image1.png)
