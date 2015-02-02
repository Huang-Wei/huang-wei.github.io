---
author: superpippo
comments: true
date: 2012-07-18 17:19:37+00:00
layout: post
slug: config_github_via_egit
title: EGit下配置Github项目
wordpress_id: 202
categories: programming
tags:
- egit
- github
---

现在这年头，Github上没有几个项目都不敢跟人打招呼了，越来越多的招聘公司把Github上的独立项目作为一项考核指标。So，如果少年你也是跟我一样：1）工作中使用Eclipse ；2）时不时有新奇的想法/代码与大家分享；3）还没有Github项目。那么，开始创建你的第一个Github项目吧。

## Step 1：安装Git
 
安装地址在[这儿](http://git-scm.com/)，安装完之后在环境变量中设置HOME为Git的bin目录（例如`D:\Program Files (x86)\Git\bin`），然后可以启动Git的shell看看，其中包含一些通用的环境设置，在windows上有点类似于cygwin。

## Step 2：在Eclipse上安装EGit

首先提一下Eclipse的版本变化，从Eclipse 3.0至今每年6月发布一个稳定版本（一般是3.X.2），到今年3.8已经是第九个年头。而3.8也将是Eclipse 3系列的最后一个版本，之后将不再开发。取而代之的是Eclipse e4项目，而今年与3.8同时发布的4.2是第一个正式版本（已经登上[http://www.eclipse.org/downloads/](http://www.eclipse.org/downloads/)的首页）。e4的更多信息详见[http://www.eclipse.org/e4/](http://www.eclipse.org/e4/)。

回到EGit本身，最新版本（2.0.0）是for Eclipse 3.8/4.2的。所以你可能需要在[Older Releases](http://wiki.eclipse.org/EGit/FAQ#Where_can_I_find_older_releases_of_EGit.3F)中寻找和你Eclipse版本匹配的EGit版本，否则会安装不上。例如，我的Eclipse环境为3.6.2，那么就应该安装EGit 1.3.0。

这里再顺带提一下Eclipse的版本选择，一般来说，偶数版本（小数点后一位）更加稳定。如果一个商业软件发布频率不算太高的话，偶数版本也更多地被商业软件采用为标准的Eclipse base。笔者从04年开始使用Eclipse，使用的版本也都是偶然版本：3.0.X，3.2.X，3.4.X，3.6.X。

## Step 3：在Github上创建repository

建一个叫Git-Test的repository，那么会得到一个https地址：`https://github.com/Huang-Wei/Git-Test.git`。

PS: 现在的Github不再仅仅支持open-ssh，对于windows用户且使用Https的方式就不用导入public key至Github。
    
## Step 4：配置EGit

安装完EGit之后，进入Preferences > Team > Git，指定默认的repository地址，如C:\Users\superpippo。Preferences > Team > Git > Configuration，这里面的键值对都记录在C:Users\superpippo.gitconfig中。

![](/images/201207/image.png)

其他均默认。

**If 第一次从本地上传代码，go to [Step 5](#step5)；Else if 从Github第一次check out代码，go to [Step 7](#step7)。**

## <a name='step5'></a>Step 5：创建本地项目并提交至本地

创建一个简单的Java Project并命名为Git-Test，右键Git-Test ，Team > Share Project... > Git

![](/images/201207/image1.png)

右键Git-Test，Commit...，选中所有文件提交。至此所有的文件都已纳入本地Git的版本控制中。

## Step 6：Push至GitHub

右键Git-Test，Team > Remote > Push...。贴入之前在GitHub创建的repository的HTTPS地址:

![](/images/201207/image2.png)

Next，选择Add All Branches Spec。

![](/images/201207/image3.png)

Next。

![](/images/201207/image4.png)

Finish。成功的话会有提示。

![](/images/201207/image5.png)

## <a name='step7'></a>Step 7：从GitHub上Check out代码

把刚才创建的Git-Test项目删除。

File > Import... > Git > Projects from Git > URI。

![](/images/201207/image6.png)

一路Next即可。

至此，project就通过EGit与GitHub关联上。

1. 有代码改动时，先Commit至本地。然后通过Team > Push to Upstream上传至Github。

1. 如果Github上有更新的代码，通过Team > Fetch from Upstream同步本地代码。

如果这两个按钮失效，以下是我的Repository Settings作为参考：

![](/images/201207/image7.png)
