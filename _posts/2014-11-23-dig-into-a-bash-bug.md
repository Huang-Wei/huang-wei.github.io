---
title: 从一个bash "bug"说起
tags: ssh bash loop
lang: zh
---

## 两个例子

假设文件`vms`里存着两个vm的名称：

```
vm1
vm2
```

那么你期待下面的脚本`while_loop.sh`会输出什么？

```bash
#!/bin/bash
# while_loop.sh
while read vm; do
  echo "$vm"
  ssh root@${vm} "sleep 2"
done < vms
```

<!--more-->

结果像下面这样：看起来vm2的结果根本就没有执行！！

```
vm1
```

那么再来看看下面这个脚本`for_loop.sh`：

```bash
#!/bin/bash
# for_loop.sh
vms=(vm1 vm2)

for vm in ${vms[@]}; do
  echo "$vm"
  ssh root@${vm} "sleep 2"
done
```

它和上面的脚本不同之处只是把`while`循环换成了`for`循环，但是输出结果一切正常：

```
vm1
vm2
```

当我碰到这个问题时，表情一开始是这样的：
![Alt text](/images/201411/kidding-me.png)

研究了一番，发现根本问题在于：**`ssh`命令默认把当前的标准输入**（很不巧，在while循环中，当前的标准输入是vms文件中第一行之后的所有内容）**作为了远程执行命令的标准输入**——尽管在这个例子中，sleep命令根本不需要标准输入。

那么，我们用`cat`命令来模拟一下远程命令需要标准输入的情况：

```bash
#!/bin/bash
# while_loop_1.sh
while read vm; do
  echo "###$vm###"
  ssh root@${vm} "cat"
done < vms
```

输出结果跟预想的一样：

```
###vm1###
vm2
```

知道了原因，怎么来改这个程序就很容易了，有三种方法：

1. 将`while`中的标准输入重定向至`/dev/null`:  
    `0</dev/null ssh root@${vm} "sleep 2"`
1. 用`/dev/null`来作为标准输入:  
    `ssh root@${vm} "sleep 2" < /dev/null`
1. 使用`ssh -n`:  
    `ssh -n root@${vm} "sleep 2"`
    > -n Redirects stdin from /dev/null (actually, prevents reading from stdin).

## Refs:

* http://www.unix.com/shell-programming-and-scripting/38060-ssh-break-while-loop.html
* http://novosial.org/shell/while-ssh/index.html
* http://linux.spiney.org/help_ssh_eating_all_my_standard_input