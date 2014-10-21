---
layout: post
title: SSH使用小结
description: 小命令里有大世界，记录SSH使用的各种技巧。
keyword: ssh, port, forwarding, tunnel
categories: programming
---

上大学的时候，曾经看到有小伙伴买过这本书（当时是第一版），当时心想：不是吧，`SSH`还能出一本书呢，不就是一命令吗？是的，虽然只是一个命令，但能用好它也绝非易事。这篇文章将从最实用的角度出发，写一些`ssh`的技巧。

## 1. 远程登录

这可能就是我上学时以为`ssh`功能的全部了，也是最基本功能。

### 1.1. ssh username@host

这是最常用的方式，输入密码就可以远程访问了，但仅限于远程ssh服务器允许密码访问的时候。

### 1.2. ssh [-i private_key] username@host

这条命令用于在远程ssh服务器不允许密码访问（只允许私钥）时，具体做法是这样的：

* 在本机用`ssh-keygen`生成一对openssh协议的钥匙对：默认的加密算法是`rsa`，存放位置是`~/.ssh/id_rsa`和`~/.ssh/id_rsa.pub`。
* 将公钥的内容`~/.ssh/id_rsa.pub`粘贴到远程服务器上的`~/.ssh/authorized_keys`（这个默认位置也可以通过`/etc/sshd/config`来修改）。当然，`ssh-copy-id`（需要额外安装这个包）能帮你做同样的事。
* 如果你的私钥是存放在默认位置和默认命名（`~/.ssh/id_rsa`），那么直接`ssh username@host`你就可以访问远程主机了。否则需要用`-i`显式指定。
* **注意1**：如果你是在windows下，千万不要把第三方工具（例如putty或securecrt）的key和标准openssh协议的key相混淆。以putty为例，你需要将openssh的私钥（`id_rsa`）用它自带的`puttygen`工具转换一下，生成一个`.ppk`文件，然后在连接的session中指定这个`.ppk`文件。
* **注意2**：在远程服务器上的`.ssh`文件夹权限最好为`700(rwx)`，权限过于开放是无法使用password-less方式的。

### 1.3. ssh host

这条命令有两层意思：

* 如果你的本地有`~/.ssh/config`这个文件，那么它会读取里面的内容，例如你可以写入以下内容：那么`ssh host`就相当于你以以下这些配置连接到了`9.9.9.9`
    
```
Host host
    HostName 9.9.9.9
    IdentityFile ~/.ssh/id_rsa
    User hweicdl
    ServerAliveInterval 15
    Port 12345
    ForwardAgent yes
```

* 另一层意思就是这条命令省略了用户名：它会以当前的登录名访问远程服务器，相当于`ssh $(whoami)@host`

## 2. 端口转发

上面灌了好些水，目的是为了引出这一节的内容，这是这篇文章最想写的，也是我认为最有价值的东西。`ssh`的连接相当于一个加密的安全通道（tunnel），透过它能做很多有意思的事情。

### 2.1. SSH -L

本地端口转发，使用方式为：

```
ssh -L [local_port]:[remote_host]:[remote_host_port] username@ssh_server
```

意思就是：在本地与`ssh_server`间建立起一条ssh连接通道，它监听本地的`local_port`端口，所有到这个端口的请求都透过ssh通道转发到`remote_host`的`remote_host_port`上去。

假设你有一个openstack集群（vm1 ~ vmx），它们仅能从jumpbox访问，而jumpbox与本地网络连通。那么下面这条命令就建立了一条本机到jumpbox的ssh通道，并借由此将本地的请求转发到vm1上。

```
ssh -L 8000:vm1:80 username@jumpbox
```

* **注意1:** 这条命令成功的前提是vm1的80端口是能从jumpbox直接访问的。如果vm1的80端口是在防火墙后面的，那就要做两层转发：`本地:8000 -> jumpbox:[some_port]` and `jumpbox:[some_port] -> vm1:80`。即：

    ```
    ssh -L 8000:localhost:7777 username@jumpbox # 此命令运行在本地
    ssh -L 7777:localhost:80 username@vm1 # 此命令运行在jumpbox
    ```

* **注意2:** 如果想在后台长时间地维持这个通道，可以加入以下参数：`ssh -qTfnN -L 8000:vm1:80 username@jumpbox`。

`SSH -L`可以理解为“正向”代理：由本地到远程；而`SSH -R`就可以理解为“反向”代理。

### 2.2. SSH -R

远程端口转发，使用方式为：

```
ssh -R [ssh_server_port]:[remote_host]:[remote_host_port] username@ssh_server
```

与本地转发有几点不同：

* 这里的`remote_host`是本地机器可以访问的某个主机。而本地端口转发中，`remote_host`是`ssh_server`能访问的主机。
* 还是以openstack集群为例，大部分情况下，jumpbox只是和本机在一个subnet，是无法反向地访问本地网络及公网的，那么就无法只靠一条ssh命令来让vm1访问到本地网络及公网，必须做两层反向代理：`jumpbox:[some_port2] <- vm1:[some_port3]` and `[remote_host]:[some_port1] <- jumpbox:[some_port2]`，即：

    ```
    ssh -R 8888:localhost:7777 username@vm1 # 此命令运行在jumpbox上
    ssh -R 7777:remote_host:9999 username@jumpbox # 此命令运行在本地
    ```

  这样在vm1上就可以访问其8888端口来访问remote_host的9999端口。

我经常把反向代理用在下列的场景中：

* 公司内网防火墙后的机器，一般是不能访问公网的。那么起一个反向代理，就可以让它们访问公网。
* 如果访问的外网是github，端口是22，那么在公司内网的机器就可以直接操作github上的项目。

    ```
    ssh -R 6666:github.com:22 username@intranet_vm # 运行在你可以访问公用的pc
    git clone ssh://git@localhost:6666/<github_project> # 运行在intranet_vm
    ```

* 一般公司都有不关机的台式机，基本是不关机的。那么在其上起一个反向代理到某个你的云主机（softlayer，阿里云什么的，有公网ip并且开了ssh权限就行），然后在家里登上你的云主机，就可以访问公司的内网了。

### 2.3. SSH -D

如果你有一台在国外的云主机，或者你的公司在国外有机器可供使用，那么你可能会想用下面的方式做些“中国特色”的事情了：

```
ssh -L 8000:www.google.com:80 username@vm_outside_greatwall
```

这样在浏览器里输入`localhost`就可以访问google了，不过如果你要访问别的被“墙”的网站呢？难道每一个网站都要起一个ssh？

当然不用，`SSH -D`就可以做到：`D` means `a local "dynamic" application-level port forwarding`。其在本地分配了一个socket用于监听某端口，并将通过此端口的连接由ssh tunnel转发。这个ssh tunnel相当于一个socks server。

例如在本地运行：

```
ssh -D 7000 username@vm_outside_greatwall
```

然后在chrome中的proxy settings就可以指定localhost:7000作为socks server以访问各类被“墙”的网站。不过基本没人在原生的proxy settings里配，很多插件很方便地配置和切换，`SwichySharp`就是其中之一。另外，`proxychains`可以在命令行下控制每一条命令，应用程序都由socks server连接。

## 3. SSH远程调用

## 4. SCP

## 5. Misc