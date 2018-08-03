---
title: DB2安装后未监听本地端口
tags: db2 tcpip
lang: zh
---

前两天在suse 64位机器上安装DB2 10.1时发生一个奇怪的现象：安装成功，但只能服务器端操作，客户端JDBC连接，或是catalog后db2cmd的连接皆无果。

例如：在安装过程中会添加一个db2instX的帐户，及一个db2 instance，监听在5000X端口。但客户端catalog完这个node再connect总显示TCP/IP communication错误。在检查完防火墙，并`telnet XX.XX.XX.XX 5000X`后确定客户端及网络没有问题。

那么就应该是服务器安装后的一些冲突或bug：

<!--more-->

## 1. 查看当前实例配置信息中的SVCENAME值

![](/images/201208/image.png)

可以看到TCP service name为db2c_db2inst5

## 2. 检查是否监听某个端口

```
netstat -a | grep "db2c_db2inst5"
```

如果未返回结果则可以证明此实例未监听任何端口。

PS：一个正常监听的端口在netstat -a及/etc/services中的定义应该是这样的：

![](/images/201208/image1.png)

![](/images/201208/image2.png)

## 3. 查看db2set

```
db2set
```

会发现DB2COMM=TCPIP这一项值应该未设置。添加此项值：

```
db2set DB2COMM=TCPIP
```

## 4. 重启db2：`db2stop & db2start`
