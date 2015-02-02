---
layout: post
title: lifebackup.cn域名过期
keyword: ssh, bash, loop
categories: programming
description: lifebackup.cn域名过期了>_<
---

## 1) lifebackup-cn

`lifebackup.cn`是我在研一的时候申请的，虽然跟着我有7年的历史了，但始终纠结着这不是个顶级域名，so上个月到期后就没有再续费了>_<

在跟着`huang-wei.github.io`这个二级域名厮混前，还需要解决图片URL的迁移工作——之前的图片都存在cowoo的主机上，想着直接`sed/www.lifebackup.cn/<ip>/`就OK了，但ip `173.192.191.35`直接指向的是cowoo的主站`productivelife.cn`。就不麻烦cowoo去更新域名，配apache的virtual host了，况且访问速度也一般，于是准备把所有图片直接放到github上。

## 2) 备份wordpress

这个过程用的是[exitwp](https://github.com/thomasf/exitwp)——一个把workpress文章备份成jekyll格式的项目。虽然最终生成的markdown语法还不够智能，但已经相当好用了，尤其是会把文章中依赖的所有图片单独存到本地（默认运行不存储图片）。

运行后的目录结构大致是这样：

```
exitwp
├── build
│   └── jekyll
│       └── www.lifebackup.cn
│           ├── images
│           │   ├── 2005-01-10-e58f88e8a781e7a791e696afe5a194e5ba93e5a194
│           │   ├── 2005-04-05-superpippo-allusion
│           │   │   └── superpippo_thumb.jpg
│           │   ├── 2006-07-27-e8888de793a6e5a5bde8bf90
│           │   │   └── x1pNWjjkHJ3o_xzNrSiAUrfJARfIhcF_-9lcaR7df-1A_Z
│           │   ├── 2007-07-05-mylyne8af95e794a8e7ac94e8aeb0
│           │   │   ├── y1p1VfjBGItETiJ5feio7adZiKklo6DAMJ3FtISoquIZtdku5nI4DyUo0FTeP4ud3N7
│           │   │   ├── y1p1VfjBGItETj5l4Um0JAi2EG8HZdLgBUSsKfPNn3l0S_LNMEEtyxQBsHQ7m7oMDf0
│           │   ├── ......
│           └── _posts
│               ├── 2005-01-10-e58f88e8a781e7a791e696afe5a194e5ba93e5a194.markdown
│               ├── 2005-01-18-e7a6bbe6a0a1e5898de590ace79a84e6bc94e8aeb2.markdown
│               ├── 2005-03-06-first-step-to-linux.markdown
│               ├── ......
├── config.yaml
├── exitwp.py
├── html2text.py
├── html2text.pyc
├── pip_requirements.txt
├── README.rst
├── Vagrantfile
└── wordpress-xml
    └── superpippo.lifebackup.wordpress.2013-12-13.xml

26 directories, 167 files
```

## 3) jekyll update

我在本地装了个jekyll，在测试blog的时候很方便。更新blog图片URL的过程大致是：

1. 找出blog中所有含图片的文章
2. 把图片拷贝到jekyll相应的images文件夹中
3. 把文章中的图片URL更新为相应image的路径（相对路径）

写了一个小脚本来做这个事：

<script src="https://gist.github.com/Huang-Wei/93416f4f237bad150733.js"></script>

## 4) 处理一些特殊情况

上面的脚本会把一些无法匹配的情况打印出来，基本都是[exitwp](https://github.com/thomasf/exitwp)没有生成标准的markdown图片格式导致的，需要手工操作一下。

## 5) push to github

最后push到github就OK了，应该所有文章的图片都不会404了。