---
author: superpippo
comments: true
date: 2013-05-24 05:09:49+00:00
layout: post
slug: ubuntu-1204-samba-config
title: Ubuntu 12.04 Samba配置小记
wordpress_id: 510
categories: programming
tags:
- linux
- samba
---

本来这种文章网上一搜一大把，也没什么记的必要，但是亲身实践过后，发现有些细节还是记录下来的好，估计半年之后也就全忘了。

主要步骤跟[这篇文章](http://wiki.ubuntu.org.cn/Samba)说的并无太多不同，基本一路配下来不会有什么问题。主要想说的是这几个小tip：

## 1) 关于useradd和adduser

对于ubuntu来说，`useradd`不会在本地建立`/home/${user}`文件夹，也不会赋本机登录密码（除非你之后显式地`passwd ${user}`），也就是说useradd完的用户是**不能从本机登录**的。  
而adduser会一次性地赋本机密码，创建完的用户是正常的本地用户。  
对于samba共享的用户来说，`useradd`就足够了。

## 2) 关于NT_STATUS_LOGON_FAILURE

配完samba，一般会`smbclient –L //localhost`测试一下。而随之而来可能会遇上NT_STATUS_LOGON_FAILURE错误。这个问题网上大部分的讨论是基于你的samba用户没有设置密码（smbpasswd），但我在实践的时候明明设置过smb密码，却仍然出现这个错。  
原因是`smbclient –L //localhost`默认的连接用户是你当前用户，而通过useradd的用户是不能通过su -切换的，你必须显式指定samba用户来检查连接：`smbclient –U ${smbuser} –L //localhost`。  
PS：对于windows客户端，可以用`net view \\{smbserver}`来检查连通性。

##3) 防火墙  

这个问题卡了好久好久。。我用的是我司基于Ubuntu12.04的定制Linux，也不知道是我司默认配了些iptables rule，还是Ubuntu本身就带。导致一切配置都正常，仍然无法连接到samba服务器。最后简单粗暴把iptables服务停掉，待有时间再研究下iptables。

## 4) 误删smb.conf  

由于被#3折腾了好久，smb.conf被改的面目全非。而在`apt-get remove`的时候没有`--purge`（其实--purge也不会删这个文件），心想就干脆删掉再重装吧。这下悲剧了，重装的samba根本不会恢复这个配置文件，也就根本没法启动。Google了下，尝试了神马`-o DPkg::Options::="--force-confmiss"`，`-o DPkg::Options::="--force-confnew"`都没用。最后没辙，从别地儿拷了一个过来。  
其实并不是purge或是dpkg的参数不好使，而是因为这些命令所说的配置文件都是指的samba软件包的**元**配置文件（`/etc/default/samba`），而smb.conf这个文件在你不装samba之前就已经存在于`/etc/samba`里，所以以为重装samba能帮你恢复这个文件，纯粹是自己骗自己了。so骚年们，养成改配置文件前先备份的习惯吧。。
