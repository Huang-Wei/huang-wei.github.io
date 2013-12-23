---
layout: post
title: 在本地kvm运行CloudInit
description: CloudInit是最先由Amazon贡献，用于EC2的instance初始化，继而被各linux厂商（[ubuntu](https://help.ubuntu.com/community/CloudInit)，[redhat](https://rhn.redhat.com/errata/RHEA-2013-0535.html)）所采纳，用在各种linux的cloud image上。这篇文章基于Ubuntu 12.04，谈谈如何使用CloudInit以NoCould的方式初始化linux image，以及这种方式对我们可能有的启发。
keyword: linux, cloudinit, kvm
categories: programming
---

CloudInit是最先由Amazon贡献，用于EC2的instance初始化，继而被各linux厂商（[ubuntu](https://help.ubuntu.com/community/CloudInit)，[redhat](https://rhn.redhat.com/errata/RHEA-2013-0535.html)）所采纳，用在各种linux的cloud image上。这篇文章基于Ubuntu 12.04，谈谈如何使用CloudInit以NoCould的方式初始化linux image，以及这种方式对我们可能有的启发。

## 安装KVM

首先按照[这篇文章](https://help.ubuntu.com/community/KVM/Installation)安装好KVM。

```
hweicdl@hweicdl-t430:~$ lsmod | grep kvm
kvm_intel             137888  0 
kvm                   422160  1 kvm_intel
hweicdl@hweicdl-t430:~$ kvm-ok
INFO: /dev/kvm exists
KVM acceleration can be used
```

如果无法安装很有可能是被BIOS禁掉了，那么需要在BIOS里打开：

![](/images/201312/bios.jpg)

## 安装cloud-init

- 对于12.10之后的版本，只需要 `sudo apt-get install cloud-utils`。
- 由于`cloud-localds`是在ubuntu 12.10之后才加入的，对于12.04，除了上面的命令，还需要手工安装`cloud-localds`：

```
bzr branch lp:cloud-utils
sudo cp cloud-utils/bin/cloud-localds /usr/local/bin
sudo apt-get --assume-yes install genisoimage qemu-utils coreutils
```

## 下载ubuntu cloud image

我们选用13.10的[cloud image](http://cloud-images.ubuntu.com/releases/13.10/release/)下载：

`wget http://cloud-images.ubuntu.com/releases/13.10/release/ubuntu-13.10-server-cloudimg-amd64-disk1.img`

下载回来的是200多M的压缩qcow2格式，当然可以直接来用，只不过在启动的时候会做解压，影响启动速度，在这儿我们把它解压：`qemu-img convert -O qcow2 ubuntu-13.10-server-cloudimg-amd64-disk1.img my_vm.img`

之后，我们就都用这个解压后的`my_vm.img`：

```
hweicdl@hweicdl-t430:~/projects/cloudinit/test$ qemu-img info my_vm.img 
image: my_vm.img
file format: qcow2
virtual size: 2.0G (2147483648 bytes)
disk size: 825M
cluster_size: 65536
```

## cloud-init配置脚本

cloud-init的配置脚本可包含多种[类型](https://help.ubuntu.com/community/CloudInit)，其中最重要的是cloud config data，即配置密码，ssh key，网络，mount point等等，完整的信息参考[link1](http://bazaar.launchpad.net/~cloud-init-dev/cloud-init/trunk/view/head:/doc/examples/cloud-config.txt)，[link2](http://cloudinit.readthedocs.org/en/latest/topics/examples.html)。

在这我们只选取部分信息来构建cloud config data：

```
hweicdl@hweicdl-t430:~/projects/cloudinit/test$ cat my-user-data 
#cloud-config
password: passw0rd
chpasswd: { expire: False }
ssh_pwauth: True

ssh_authorized_keys:
 - ssh-rsa 你的~/.ssh/id_rsa.pub内容

timezone: Asia/Chongqing
```

有了这部分信息，就可以构建一个no-cloud的metadata image（否则的话也能启动vm，但内部会一直尝试连接169.254.169.254几分钟）：`cloud-localds my-seed.img my-user-data`。

接着就可以以no-cloud的方式启动这个vm：`kvm -net nic -net user,hostfwd=tcp::2222-:22 -hda my_vm.img -hdb my-seed.img -m 512`。

由于作了ssh port forward，除了有原生的QEMU窗口登录外，还可以直接ssh登录：`ssh -p 2222 ubuntu@localhost`。

## Go deep in cloud-init

先来看看`cloud-localds`的help信息：

```
hweicdl@hweicdl-t430:~/projects/cloudinit/test$ cloud-localds help
Usage: cloud-localds [ options ] output user-data [meta-data]

   Create a disk for cloud-init to utilize nocloud

   options:
     -h | --help            show usage
     -d | --disk-format D   disk format to output. default: raw
     -H | --hostname    H   set hostname in metadata to H
     -f | --filesystem  F   filesystem format (vfat or iso), default: iso9660

     -i | --interfaces  F   write network interfaces file into metadata
     -m | --dsmode      M   add 'dsmode' ('local' or 'net') to the metadata
                            default in cloud-init is 'net', meaning network is
                            required.
     -v | --verbose         increase verbosity

   Note, --dsmode, --hostname, and --interfaces are incompatible
   with metadata.

   Example:
    * cat my-user-data
      #cloud-config
      password: passw0rd
      chpasswd: { expire: False }
      ssh_pwauth: True
    * echo "instance-id: $(uuidgen || echo i-abcdefg)" > my-meta-data
    * cloud-localds my-seed.img my-user-data my-meta-data
    * kvm -net nic -net user,hostfwd=tcp::2222-:22 \
         -drive file=disk1.img,if=virtio -drive file=my-seed.img,if=virtio
    * ssh -p 2222 ubuntu@localhost
must provide output, userdata
```

可以看到`cloud-localds`还可以接收一个可选的参数：`meta-data`。这个文件里的内容是与instance相关的，例如hostname，instance-id等。一般情况下（也是绝大多数情况）它都应该只用来做初始化，即运行完一次就不要再修改它了。只要instance-id不变，即使以`kvm -net nic -net user,hostfwd=tcp::2222-:22 -hda my_vm.img -hdb my-seed.img -m 512`启动N遍，也再不会使用任何user-data和meta-data中的内容。这也是`cloud-init`这个词中`init`的意义，它只需要也应该run once。

但从另一个角度看，我们也可以以重新生成instance-id的方式去“强行”重新初始化这个cloud image/instance。即启动时像上面的`uuidgen`就重新生成一个id，然后用`cloud-localds`去重新生成`my-seed.img`。

另外，上面说过cloud config data是最重要且必需的一类数据，若想加入其他初始化行为，就必须把所有的数据以MIME的方式写在同一个文件里。例如我们有一个shell，它在启动时echo一句话到某个文件：

```
hweicdl@hweicdl-t430:~/projects/cloudinit/test$ cat hello_world.sh 
#!/bin/bash
echo "hello world!" >> /home/ubuntu/test
```

我们可以把这种行为与cloud config data合并在一起：`write-mime-multipart --output=combined-userdata.txt hello_world.sh:text/x-shellscript my-user-data`。然后重新生成no cloud data：`cloud-localds my-seed.img combined-userdata.txt`，最后启动kvm即可。