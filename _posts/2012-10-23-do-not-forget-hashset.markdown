---
author: superpippo
comments: true
date: 2012-10-23 12:53:56+00:00
layout: post
slug: do-not-forget-hashset
title: 别忘了HashSet
wordpress_id: 233
categories:
- programming
tags:
- hashset
- java
---

由于长期间的思维惯性和对Java程序缺乏足够的性能重视，导致我在处理Java集合类时经常想当然的只使用List。

例如经常有这样的场景：给定一个集合M，要遍历其中的每一个元素是否在集合N中（需要自己创建）是否存在。那么可能就会机械性地写出以下代码：

```java
// construct the collection N       
List<MyObject> list_N = new ArrayList<MyObject>();        
for (int i = 0; i < N; i++) {        
	// construct or get an Object[i]        
	list_N.add(MyObject_i);        
} 
// check the element in M is in collection N or not       
for (Object o : collection m) {        
	if (list_N.contains(o))        
		...        
	else        
	...   
}
```

乍一看也没太大问题，但是当N很大时，这样的写法就有性能上的隐患——因为List的contains()实现是遍历式的匹配，也就是说上述的代码的时间复杂度是O(M*N)。

而如果用HashSet来构造，在之后的contains()调用时就是哈希查找，只需要O(1)的复杂度，总的时间复杂度为O(M)。

下面用字符串做为测试对象（采用字符串默认的哈希函数）来看看这两者的实践耗时：

```java
public static void main(String[] args) {       
	List<String> list = new ArrayList<String>();        
	Set<String> set = new HashSet<String>();        
	for (int i = 0; i < 10000; i++) {        
		String s = "t" + i;        
		list.add(s);        
		set.add(s);        
	}        
	long time1 = System.currentTimeMillis();        
	for (int i = 0; i < 100; i++) {        
		String s = "test" + i;        
		if (set.contains(s))        
			System.out.println(i);        
	}        
	long time2 = System.currentTimeMillis();        
	System.out.println("Spend " + (time2 - time1));        
	for (int i = 0; i < 100; i++) {        
		String s = "test" + i;        
		if (list.contains(s))        
			System.out.println(i);        
	}        
	long time3 = System.currentTimeMillis();        
	System.out.println("Spend " + (time3 - time2));        
}
```

在M为100，N为10000的测试场景中，这两者的时间差别在我机器上差了1~2个数量级。

所以，在这种集合数量很大的时候，应该率先想到hash。而在设计一些方法和接口的时候，多使用Collection，而不是直接设计成List，会给程度实现带来更大的自由度。
