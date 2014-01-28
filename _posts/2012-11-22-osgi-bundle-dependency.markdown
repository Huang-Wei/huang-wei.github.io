---
author: superpippo
comments: true
date: 2012-11-22 09:22:38+00:00
layout: post
slug: osgi-bundle-dependency
title: Let's Talk About OSGi Bundle Dependency
wordpress_id: 246
categories: programming
tags:
- osgi
---

## 1. OSGi Terms

* OSGi, Equinox and Eclipse  
从历史上来说，OSGi和Eclipse一开始都是独立发展的。OSGi首先是一套规范，核心在于它模块化，松耦合的理念，而Eclipse致力于构建一套基础的工具平台。         
在2003年，基于Eclipse出现了Eqinox项目：其最初的目的是解决当时Eclipse的一些运行时问题，而OSGi以清晰的组件模型，强大的执行规范为选为最可能的实现参考。         
之后就是两个组织间理念的碰撞及融合，Equinox借鉴了很多OSGi的思想，反过来也影响了OSGi的发展。OSGi的R4版本就是二者相互影响后的一个结果。至今，Equinox依然是OSGi规范的参考实现(Reference Implementation)。         
而在Eclipse这边，其运行时在3.1后就被替换成Equinox——完全基于OSGi的运行时实现。 
   
* Plugin and Bundle        
二者在概念上来说，其实都是「模块」的同义词。只不过从历史上来说，OSGi喜欢用Bundle这个词，Eclipse则更喜欢用Plugin这个词。         
而Plugin作为一种特殊的Bundle，特殊在其有一个特殊的描述文件：plugin.xml。 
   
* plugin.xml and MANIFEST.MF        
在Eclipse的3.0或之前版本，是只有plugin.xml的。因为如前面说到的，从3.1开始其运行时才完全开始采用Equinox。         
而在Eclipse完全基于OSGi后，二者的差别就比较小了。              

* MANIFEST.MF是作为最核心的组件描述符存在的：Bundle Version, Name, Dependency等等都在其中定义 
       
* plugin.xml是作为MANIFEST.MF的一个补充。除了OSGi的标准Service模型，Equinox还有一套自己的机制：Equinox Extension Registry。也就是我们常用的Eclipse扩展点机制，在功能上它是对OSGi的一个很好补充。 

## 2. Bundle Dependencies

`Import-Package`和`Require-Bundle`是说明Bundle间依赖的两个关键字，二者以不同粒度指定了Bundle间的依赖关系，它们都在MANIFEST.MF中定义。

* `Import-Package`：package是细粒度的模块划分。例如A bundle将_x.y.z.common_和_x.y.z.core_利用Export-Package关键字暴露出来，其他的Bundle就可以利用Import-Package引用common或core。

* `Require-Bundle`：相对于bundle是粗粒度的模块划分。例如A bundle有_x.y.z.common_，_x.y.z.core_和_x.y.z.internal_三个package，并利用Export-Package关键字将common和core包暴露出来。那么当B bundle指定_Require-Bundle:A_时，B就可以访问common和core包，但不包括internal包，因为A并未将其Export出来。

一般来说，`Import-Package`的控制粒度更精确，经常被那些一开始就使用OSGi的开发者所使用，这也对整个系统的设计上提出了更高要求。而`Require-Bundle`似乎更经常被那些一开始就使用Eclipse的开发者所使用，粒度会更粗。虽然在实际上开发过程被大量使用，但按照OSGi学院派的说法是不被推荐使用的。（这个问题就见仁见智了，暂不在本文的讨论范围内）

## 3. VERSION

每个OSGi中的Bundle，Bundle中各个Export的package均需要指定version，如不指定默认为0.0.0。这些version会在被其他Bundle依赖时（Import-Package或Require-Bundle）用到。
  
* Bundle级别的版本依赖        
假设有两个Bundle，其Bundle-SymbolicName均为A（Bundle-Name分别为A1和A2），其Bundle-Version为1.0.0和2.0.0。
![image](http://www.lifebackup.cn/wp-content/uploads/2012/11/image.png)
此时B bundle依赖A bundle（不指定依赖的具体版本），那么在运行时B所依赖的package均来自A2。因为OSGi默认取依赖的最高版本，此例中为2.0.0。         
按照这个原则，指定具体的依赖为[1.0.0, +∝)或[1.0.0, 2.0.0]时均会取A2的package(2.0.0)。         
只有在依赖为类似[1.0.0, 2.0.0)时，B bundle才会去找A1的package(1.0.0)。 
   
* Package级别的版本依赖        
在上例的基础上，假设A1 Export的package为_x.y.z.common(1.0.0)_和_x.y.z.core(1.0.0)_，A2 Export的package为_x.y.z.common(2.0.0)_和_x.y.z.core(2.0.0)_。         
此时B bundle不依赖任何bundle，只在Import-Package中指定_x.y.z.common_与_x.y.z.core_：
![image](http://www.lifebackup.cn/wp-content/uploads/2012/11/image1.png)
依赖的规则也遵循数学上的开闭区间规则，即依赖的是common和core的2.0.0版本。但Import-Package是package级别的依赖，并不需要绑定于某个Bundle实现，这样就更灵活。如：
![image](http://www.lifebackup.cn/wp-content/uploads/2012/11/image2.png)
就采用了_x.y.z.common_的1.0.0版本（属于A1 bundle），以及_x.y.z.core_的2.0.0版本（属于A2 bundle）。
   
* Bundle依赖与Package依赖冲突        
假设Bundle B指定Require-Bundle: A;bundle-version="[2.0.0, +∝)"，但同时又指定  
```
Import-Package: x.y.z.common [1.0.0, 2.0.0), x.y.z.core [1.0.0, 2.0.0)
```  
。这种冲突情况下，仍然会采用common和core的1.0.0版本。即以Import-Package为准，即使Require-Bundle Export了更高版本的package。OSGi规范的3.9.4节有详细阐述。
![image](http://www.lifebackup.cn/wp-content/uploads/2012/11/image3.png)

* 多个Bundle或Package版本相同  
老实说这个命题是比较eggache的，囧。。。因为没人傻到犯这种错误。但据我测试来看，如果多个Bundle版本相同，那么OSGi只会load某一份Bundle。而版本相同的package被别的Bundle Import时，import的package也是不确定的。

小技巧：在Eclipse的Plug-in MANIFEST Editor的Dependencies页，左下角有个叫做`Automated Management of Dependencies`的功能。它的意思是当你需要依赖外部package时，可以在此处先加入依赖的Bundle，然后点击”add dependencies”，相应依赖的package就会自动出现在Imported Packages Section中。      
PS：把`Preferences > Plug-in Development > Update stale manifest files priors to launching`勾上，每次代码改动时就会自动计算一遍依赖关系，也就相当了手动点了”add dependencies”。

## 4. The "Uses" Directive

当多个Bundle Export同名的package时，在运行时期依赖间会有一些潜在问题。我们举个例子来说明：

Bundle A1 export的package为x.y.z.core(1.0.0)；Bundle A2 export的package为x.y.z.core(2.0.0)。

Bundle B Import-Package:x.y.z.core;version="[1.0.0,2.0.0)"，且Export-Package: u.v.w.common。在u.v.w.common中的Demo.java代码是这样的：
![image](http://www.lifebackup.cn/wp-content/uploads/2012/11/image4.png)

因为B明确指定了依赖package x.y.z(1.0.0)，所以很显然这里的getCore()使用的x.y.z(1.0.0)中的Core.java，而不是x.y.z(2.0.0)中的。

So far so good.

我们再加入Bundle C，Import-Package: u.v.w.common, x.y.z.core。目前的依赖关系是这样的：

![image](http://www.lifebackup.cn/wp-content/uploads/2012/11/image5.png)

在C的`Activator#start(BundleContext context)`中如果像这样返回了Core，会发生什么情况呢？

```java
Core core = new Demo().getCore();
```

进一步，如果继续调用core对象的方法，又会如何呢？

```java
core.print();
```

答案是第一步的调用不会出错，而第二步的调用会报运行时的错误：

```java
java.lang.LinkageError: loading constraint violation
```

这其实是跟OSGi的类加载机制有关，每个Bundle都是由不同的类加载器加载。在运行期前，OSGi容器中C的类加载器会加载x.y.z.core(2.0.0)，而在运行期时，Core core = new Demo().getCore();这条语句的后端逻辑首先是B Bundle将x.y.z.core(1.0.0)中的Core对象返回，然后将这个1.0.0的Core对象强制转换成2.0.0的Core对象。所以在调用core.print()时会出现Runtime Type Mismatch的问题。

而为了解决这类问题，OSGi引用了`uses:=`关键字。对于这个例子在B Bundle的的MANIFEST.MF修改如下即可解决问题：

```java
Export-Package: u.v.w.common;uses:="x.y.z.core"
```

更详细的uses关键字说明见[这篇](http://blog.springsource.com/2008/10/20/understanding-the-osgi-uses-directive/)。

##5. "Uses" Conflict Problem

随着`Uses`关键字的引入，也带来了一些package使用上的冲突问题。

看完上面的例子，细心的读者可能发现一个比较tricky的问题：C Bundle所import的package x.y.z.core到底是1.0.0还是2.0.0呢？按理说不加版本默认是按照[0.0.0, +∝)匹配最高的版本，但是加了uses关键字后其实绑定的是1.0.0版本。如下图所示：

![image](http://www.lifebackup.cn/wp-content/uploads/2012/11/image6.png)

不信的话，将上面的例子中C Bundle的MANIFEST.MF修改一下：

```
Import-Package: u.v.w.common, x.y.z.core;version="2.0.0"
```

重新启动OSGi容器，你会发现如下错误：

```
The bundle "C_1.0.0 [67]" could not be resolved. Reason: Package uses conflict: Import-Package: u.v.w.common; version="0.0.0"
```

而C bundle的状态也仅仅是`INSTALLED`：

![image](http://www.lifebackup.cn/wp-content/uploads/2012/11/image7.png)

也就是说：当C间接使用了1.0.0的common包（B Bundle在Export common包时使用了uses关键字）时，同时引入2.0.0的common包是不允许的，这会造成C Bundle不能被正确加载。

这篇[文章](http://blog.springsource.org/2008/11/22/diagnosing-osgi-uses-conflicts/)详细介绍了如何调试这种package冲突的问题。

## 6. Re-Exporting

re-exporting (visibility:=reexport) 能将其require的bundle中export的包重新export，就好像他们是由当前bundle export出来的一样。这样的好处是：假设B requires A（A export _core_ package）并重新re-export A bundle。这样C require bundle B就相当于import了_core_ package。从某种角度上来说，B在这里充当了「Facade」。

在Eclipse API设计中，Eclipse UI require且re-export了JFace和SWT的bundle。

但re-exporting的过度使用，将是滋生"uses" conflict问题的温床。

## 7. 后记

这篇文章的初衷是解决在工作中遇到的OSGi环境下包冲突的问题，并利用这篇blog将它的root cause及相关概念整理一下。以下是调试这类问题的一些小技巧：

* 在Eclipse环境下，在eclipse.ini中加入`-console`来打开OSGi的控制台 
   
* 在OSGi控制台下，输入help能看到所有的命令说明。除了常用的`ss`，`start`，`stop`外，个人觉得非常重要的还有`diag`，`packages`，`header`等命令 
   
* 快捷键Ctrl+Shift+A快速定位到某个bundle和package 
   
* Plug-in Dependencies视图 
   
* 修改OSGi源代码打印出更详细的出错信息，详见这篇[文章](http://www.talendforge.org/wiki/doku.php?id=dev:troubleshooting_rcp_application)
 

## 8. References

* [http://www.osgi.org/Specifications/HomePage](http://www.osgi.org/Specifications/HomePage)       
* [http://blog.springsource.com/2008/10/20/understanding-the-osgi-uses-directive/](http://blog.springsource.com/2008/10/20/understanding-the-osgi-uses-directive/)       
* [http://njbartlett.name/2011/02/09/uses-constraints.html](http://njbartlett.name/2011/02/09/uses-constraints.html)       
* [http://blog.springsource.org/2008/11/22/diagnosing-osgi-uses-conflicts/](http://blog.springsource.org/2008/11/22/diagnosing-osgi-uses-conflicts/)       
* [http://www.talendforge.org/wiki/doku.php?id=dev:troubleshooting_rcp_application](http://www.talendforge.org/wiki/doku.php?id=dev:troubleshooting_rcp_application)       
* [http://osgified.blogspot.com/2011/10/bundle-dependencies-resolution.html](http://osgified.blogspot.com/2011/10/bundle-dependencies-resolution.html)

本篇文章的例子[在此](http://www.lifebackup.cn/wp-content/uploads/2012/11/OSGi-Article-Demo.zip)下载。
