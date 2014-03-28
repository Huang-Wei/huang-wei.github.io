---
author: superpippo
comments: true
date: 2012-12-03 14:24:35+00:00
layout: post
slug: java-7-zipoutputstream
title: Java 7 ZipOutputStream
wordpress_id: 256
categories:
- programming
tags:
- java
- java7
---

在Java 6的`java.util.zip.ZipOutputStream#finish()`实现中，若`ZipEntry`为空，会抛出异常：

```java
if (xentries.size() < 1) {          
	throw new ZipException("ZIP file must have at least one entry");           
}
```

也就是说，当你创建一个ZipOutputStream但不设置任何ZipEntry，然后持久化到文件的时候，在Java 6中会报错。

可能你会偷懒式的在代码中依赖这个Exception来作一些逻辑判断。但Java 7的ZipOutputStream实现会让你尝到苦头的。。。因为在Java 7中，上面的那一段代码（If判断）被华丽丽的去掉了。

那么，在Java 7中一个空ZipOutputStream持久化到文件中是长什么样呢？是一个大小为0的空文件吗？

我们来作个实验，代码很简单：

```java 
ZipOutputStream stream = new ZipOutputStream(new BufferedOutputStream(         
		new FileOutputStream(new File("E:/test.zip"))));          
if (stream != null) {          
	try {          
		stream.close();          
	} catch (IOException e) {          
		e.printStackTrace();          
	}          
}
```

而生成的这个test.zip有1KB的大小，用Notepad++打开显示如下：

![image](http://www.lifebackup.cn/wp-content/uploads/2012/12/image.png)

[Google](http://www.digitalpreservation.gov:8081/formats/fdd/fdd000354.shtml)了一下，这是一个标准空ZIP文件开始处的签名：

```
Hex: 50 4B 05 06       
ASCII: P K enq ack
```

既然ZIP文件的标准是早就定好的，为什么Java 6不允许通过ZipOutputStream创建「空」ZIP，而Java 7允许呢？这个原因暂时还没有google到。。。
