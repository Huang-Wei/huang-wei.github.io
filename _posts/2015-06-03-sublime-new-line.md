---
title: Sublime中的new line设置
tags: editor newline sublime
lang: zh
---

先来用sublime打开一个文件：foo.txt，随便输入几行：

```text
aaa
bbb
ccc
```

<!--more-->

保存，它看起来是这样的：

![](/images/201506/sublime1.jpg)

我们写一个简单的脚本来把这个文件逐行地读出来：

```bash
while read line; do
  echo "~~$line~~"
done < ~/foo.txt
```

你期待文件中的三行都被读出来，是吗？可是事实是这样的：

```text
~~foo.txt~~
~~111~~
```

最后一行被天狗吃了？在zsh用cat能发现一些端倪：

![](/images/201506/sublime2.jpg)

注意这里的`%`表示这一行是没有换行的。举个例子：

![](/images/201506/sublime3.jpg)

所以问题出在sublime的默认保存文件方式上：它并没有给**最后一行**加上换行符。这导致了刚才的`foo.txt`的最后一行在`while read`中并不被认为是一个合法的行。

这个[链接](https://github.com/editorconfig/editorconfig/wiki/Newline-at-End-of-File-Support)解释了默认情况下，各个编辑器在保存时是否给最后一行加上换行符。

解决方式：在`~/.config/sublime-text-3/Packages/User/Preferences.sublime-settings`中加入

```
"ensure_newline_at_eof_on_save": true
```
