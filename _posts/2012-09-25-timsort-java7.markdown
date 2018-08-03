---
title: TimSort in Java 7
tags: java java7 timsort
lang: zh
---

## 1. 为什么写这篇文章

这篇文章的根源是在产品中发现了一个诡异的bug：只能在产品环境下重现，在我的本地开发环境无法重现，而双方的代码没有任何区别。最后用remote debug的方法找到异常所在：

> Exception in thread "main" java.lang.IllegalArgumentException: Comparison method violates its general contract!

Google了这个错误，是由于Java 7内置的新排序算法导致的。这才猛然想起产品的编译环境最近升级到了Java 7。

<!--more-->

## 2. 结论

在Java 6中Arrays.sort()和Collections.sort()使用的是MergeSort，而在Java 7中，内部实现换成了[TimSort](http://en.wikipedia.org/wiki/Timsort)，其对对象间比较的实现要求更加严格：

Comparator的实现必须保证以下几点（出自[这儿](http://docs.oracle.com/javase/6/docs/api/java/util/Comparator.html#compare%28T,%20T%29)）：

- a) sgn(compare(x, y)) == -sgn(compare(y, x))  
- b) (compare(x, y)>0) && (compare(y, z)>0) 意味着 compare(x, z)>0  
- c) compare(x, y)==0 意味着对于任意的z：sgn(compare(x, z))==sgn(compare(y, z))均成立  

而我们的代码中，某个compare()实现片段是这样的：

```java
public int compare(ComparatorTest o1, ComparatorTest o2) {          
    return o1.getValue() > o2.getValue() ? 1 : -1;           
}
```

这就违背了`a)`原则：假设X的value为1，Y的value也为1；那么compare(X, Y) ≠ –compare(Y, X)。  
PS: TimSort不仅内置在各种JDK 7的版本，也存在于Android SDK中（尽管其并没有使用JDK 7）。

## 3. 解决方案

- 3.1 更改内部实现：例如对于上个例子，就需要更改为

    ```java
    public int compare(ComparatorTest o1, ComparatorTest o2) {          
        return o1.getValue() == o2.getValue() ? 0 : (o1.getValue() > o2.getValue() ? 1 : -1);           
    }
    ```

- 3.2 Java 7预留了一个接口以便于用户继续使用Java 6的排序算法：在启动参数中（例如eclipse.ini）添加

    ```
    -Djava.util.Arrays.useLegacyMergeSort=true
    ```

- 3.3 将这个IllegalArgumentException手动捕获住（不推荐）

## 4. TimSort在Java 7中的实现

那么为什么Java 7会将TimSort作为排序的默认实现，甚至在某种程度上牺牲它的兼容性（在stackoverflow上有大量的问题是关于这个新异常的）呢？接下来我们不妨来看一看它的实现。

首先建议大家先读一下[这篇](http://en.wikipedia.org/wiki/Timsort)文章以简要理解TimSort的思想。

4.1) 如果传入的Comparator为空，则使用ComparableTimSort的sort实现。

```java
if (c == null) {
    Arrays.sort(a, lo, hi);
    return;
}
```

4.2) 传入的待排序数组若小于MIN_MERGE（Java实现中为32，Python实现中为64），则

- a) 从数组开始处找到一组连接升序或严格降序（找到后翻转）的数  
- b) [Binary Sort](http://www.geneffects.com/briarskin/theory/binary/index.html)：使用二分查找的方法将后续的数插入之前的已排序数组  

```java
// If array is small, do a "mini-TimSort" with no merges
if (nRemaining < MIN_MERGE) {
    int initRunLen = countRunAndMakeAscending(a, lo, hi, c);
    binarySort(a, lo, hi, lo + initRunLen, c);
    return;
}
```

4.3) 开始真正的TimSort过程：

4.3.1) 选取minRun大小，之后待排序数组将被分成以minRun大小为区块的一块块子数组

- a) 如果数组大小为2的N次幂，则返回16（MIN_MERGE / 2）  
- b) 其他情况下，逐位向右位移（即除以2），直到找到介于16和32间的一个数   

```java
private static int minRunLength(int n) {
    assert n >= 0;
    int r = 0;     // Becomes 1 if any 1 bits are shifted off
    while (n >= MIN_MERGE) {
        n |= (n & 1);
        n >>= 1;
    }
    return n + r;
}
```

4.3.2) 类似于4.2.a找到初始的一组升序数列      
4.3.3) 若这组区块大小小于minRun，则将后续的数补足（采用binary sort插入这个数组）       
4.3.4) 为后续merge各区块作准备：记录当前已排序的各区块的大小       
4.3.5) 对当前的各区块进行merge，merge会满足以下原则（假设X，Y，Z为相邻的三个区块）：

- a) 只对相邻的区块merge  
- b) 若当前区块数仅为2，If X<=Y，将X和Y merge  
- b) 若当前区块数>=3，If X<=Y+Z，将X和Y merge，直到同时满足X>Y+Z和Y>Z

```java
do {
    // Identify next run
    int runLen = countRunAndMakeAscending(a, lo, hi, c);

    // If run is short, extend to min(minRun, nRemaining)
    if (runLen < minRun) {
        int force = nRemaining <= minRun ? nRemaining : minRun;
        binarySort(a, lo, lo + force, lo + runLen, c);
        runLen = force;
    }

    // Push run onto pending-run stack, and maybe merge
    ts.pushRun(lo, runLen);
    ts.mergeCollapse();

    // Advance to find next run
    lo += runLen;
    nRemaining -= runLen;
} while (nRemaining != 0);
```

4.3.6) 重复4.3.2 ~ 4.3.5，直到将待排序数组排序完      
4.3.7) Final Merge：如果此时还有区块未merge，则合并它们

```java
// Merge all remaining runs to complete sort
assert lo == hi;
ts.mergeForceCollapse();
assert ts.stackSize == 1;
```

## 5. Demo

这一节用一个具体的例子来演示整个算法的演进过程：

**注意**：为了演示方便，我将TimSort中的minRun直接设置为2，否则我不能用很小的数组演示。。。同时把MIN_MERGE也改成2（默认为32），这样避免直接进入binary sort。

1. 初始数组为**[7,5,1,2,6,8,10,12,4,3,9,11,13,15,16,14]**        
1. <font color='blue'>寻找连续的降序或升序序列 (4.3.2)</font>         
<font color='maroon'>[1,5,7]</font> [2,6,8,10,12,4,3,9,11,13,15,16,14]         
1. <font color='blue'>入栈 (4.3.4)</font>          
当前的栈区块为[3]         
1. <font color='blue'>进入merge循环 (4.3.5)</font>          
do not merge因为栈大小仅为1         
1. <font color='blue'>寻找连续的降序或升序序列 (4.3.2)</font>         
<font color='maroon'>[1,5,7] [2,6,8,10,12]</font> [4,3,9,11,13,15,16,14]         
1. <font color='blue'>入栈 (4.3.4)</font>         
当前的栈区块为[3, 5]         
1. <font color='blue'>进入merge循环 (4.3.5)</font>         
merge因为runLen[0]<=runLen[1]         
  gallopRight：寻找run1的第一个元素应当插入run0中哪个位置（"2"应当插入"1"之后），然后就可以忽略之前run0的元素（都比run1的第一个元素小）         
  gallopLeft：寻找run0的最后一个元素应当插入run1中哪个位置（"7"应当插入"8"之前），然后就可以忽略之后run1的元素（都比run0的最后一个元素大）         
这样需要排序的元素就仅剩下[5,7] [2,6]，然后进行mergeLow         
完成之后的结果：         
<font color='maroon'>[1,2,5,6,7,8,10,12]</font> [4,3,9,11,13,15,16,14]         
1. <font color='blue'>入栈 (4.3.4)</font>          
当前的栈区块为[8]         
退出当前merge循环因为栈中的区块仅为1         
1. <font color='blue'>寻找连续的降序或升序序列 (4.3.2)</font>         
<font color='maroon'>[1,2,5,6,7,8,10,12] [3,4]</font> [9,11,13,15,16,14]         
1. <font color='blue'>入栈 (4.3.4)</font>         
当前的栈区块大小为[8,2]         
1. <font color='blue'>进入merge循环 (4.3.5)</font>         
do not merge因为runLen[0]>runLen[1]         
1. <font color='blue'>寻找连续的降序或升序序列 (4.3.2)</font>         
<font color='maroon'>[1,2,5,6,7,8,10,12] [3,4] [9,11,13,15,16]</font> [14]         
1. <font color='blue'>入栈 (4.3.4)</font>         
当前的栈区块为[8,2,5]         
1. do not merege run1与run2因为不满足runLen[0]<=runLen[1]+runLen[2]         
merge run2与run3因为runLen[1]<=runLen[2]         
  gallopRight：发现run1和run2就已经排好序         
完成之后的结果：         
<font color='maroon'>[1,2,5,6,7,8,10,12] [3,4,9,11,13,15,16]</font> [14]         
1. <font color='blue'>入栈 (4.3.4)</font>         
当前入栈的区块大小为[8,7]         
退出merge循环因为runLen[0]>runLen[1]         
1. <font color='blue'>寻找连续的降序或升序序列 (4.3.2)</font>         
最后只剩下[14]这个元素：<font color='maroon'>[1,2,5,6,7,8,10,12] [3,4,9,11,13,15,16] [14]</font>          
1. <font color='blue'>入栈 (4.3.4)</font>         
当前入栈的区块大小为[8,7,1]         
1. <font color='blue'>进入merge循环 (4.3.5)</font>         
merge因为runLen[0]<=runLen[1]+runLen[2]         
因为runLen[0]>runLen[2]，所以将run1和run2先合并。（否则将run0和run1先合并）         
  gallopRight & gallopLeft         
这样需要排序的元素剩下[13,15] [14]，然后进行mergeHigh         
完成之后的结果：         
<font color='maroon'>[1,2,5,6,7,8,10,12] [3,4,9,11,13,14,15,16]</font> 当前入栈的区块为[8,8]         
1. 继续merge因为runLen[0]<=runLen[1]         
  gallopRight & gallopLeft         
需要排序的元素剩下[5,6,7,8,10,12] [3,4,9,11]，然后进行mergeHigh         
完成之后的结果：         
<font color='maroon'>[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16]</font> 当前入栈的区块大小为[16]         
1. 不需要final merge因为当前栈大小为1         
1. 结束 

## 6. 如何重现文章开始提到的Exception

这一节将剥离复杂的业务逻辑，用一个最简单的例子（不修改TimSort.java内置的各种参数）重现文章开始提到的Exception。因为尽管google出来的结果中非常多的人提到了这个Exception及解决方案，但并没有人给出一个可以重现的例子和测试数据。另一方面，我也想从其他角度来加深对这个问题的理解。

构造测试数据的过程是个反人类的过程:( 大家不要学我。。

以下是能重现这个问题的代码：

```java
public class ReproJava7Exception {       
    public static void main(String[] args) {        
        int[] sample = new int[]        
                {0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,        
                 0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,-2,1,0,-2,0,0,0,0};        
        List<Integer> list = new ArrayList<Integer>();        
        for (int i : sample)        
            list.add(i);        
        // use the native TimSort in JDK 7        
        Collections.sort(list, new Comparator<Integer>() {        
            @Override        
            public int compare(Integer o1, Integer o2) {        
                // miss the o1 = o2 case on purpose        
                return o1 > o2 ? 1 : -1;        
            }        
        });        
    }        
}
```

## 7. Sample Code

这篇文章的所有代码可以到github：[https://github.com/Huang-Wei/understanding-timsort-java7](https://github.com/Huang-Wei/understanding-timsort-java7)下载。

## 8. References

[http://en.wikipedia.org/wiki/Timsort](http://en.wikipedia.org/wiki/Timsort)       
[http://www.geneffects.com/briarskin/theory/binary/index.html](http://www.geneffects.com/briarskin/theory/binary/index.html)       
[http://docs.oracle.com/javase/6/docs/api/java/util/Comparator.html#compare%28T,%20T%29](http://docs.oracle.com/javase/6/docs/api/java/util/Comparator.html#compare%28T,%20T%29)       
[http://www.oracle.com/technetwork/java/javase/compatibility-417013.html#source](http://www.oracle.com/technetwork/java/javase/compatibility-417013.html#source)
