---
title: 在ubuntu 1204上快速部署openstack
tags: linux openstack kvm
lang: zh
---

最近刚刚接触openstack，就琢磨着部署个环境试试。官方[文档](http://www.openstack.org/software/start/)中推荐的[DevStack](http://devstack.org/)安装方式很对我胃口，所有的安装配置都是用shell完成的，然后如其所说： 在运行`stack.sh`的时候，可以读一读这个脚本都做了些什么。

在安装成功后，不妨读一读[这篇文章](http://blog.aaronorosen.com/building-a-multi-tier-application-with-openstack/)做一个小demo，其中有几个tips：

1. 如果你用的是32位机器，那么devstack默认上传的cirros 64位image是不能正确使用的（可能跟我的硬件虚拟化支持有关）。那就上传一个32位的[image](http://download.cirros-cloud.net/0.3.1/)吧。（PS：我在一台很老的机器上配置就遇到了这个问题）  
devstack中有个`tools`目录，里面有个`upload_image.sh`可以用来方便创建image。（具体的`glance image-create`方式详见[functions源码](http://devstack.org/functions.html)）
```
tools/upload_image.sh http://download.cirros-cloud.net/0.3.1/cirros-0.3.1-i386-uec.tar.gz
```
![](/images/201401/openstack-images.png)

<!--more-->

1. 在H release中，`quantum`已经被[改名](https://wiki.openstack.org/wiki/ReleaseNotes/Havana#Key_New_Features_5)为`neutron`，所以这篇文章的命令也得做相应的替换

1. 如果你的安装环境本机IP只能设置为DHCP，也就是说，IP都会变化(机器重启什么的)。那么你最好用offline模式来运行openstack：在`localrc`中加一行`OFFLINE=True`

1. 配置成功后，各种openstack进程会在后台运行，但它们并不是service，所以机器重启的时候，或者某些原因它们挂掉的时候，使用`rejoin-stack.sh`来重启挂掉的openstack后台进程（例如在你机器重启之后）