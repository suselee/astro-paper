---
author: Suse
pubDatetime: 2024-12-05T16:36:00Z
title: Freebsd学习笔记2
featured: true
draft: false
tags:
  - 技术
  - 运维
  - 操作系统
description: Freebsd bhyve
---

前段时间实践了FreeBSD-jail相关的一些知识点。在实践的过程中我止不住的惊叹jail的设计理念，除了生态之外，其他方面自认为比docker要好。

而关于bhyve虚拟机相关的内容，我也已经实践过并结束了，同样的在实践的过程中并没有遇到太多的坑，本篇笔记就介绍一下使用vm-bhyve在Freebsd14.1上搭建debian虚拟机的全过程。


> [vm-bhyve](https://github.com/churchers/vm-bhyve)是一款freebsd专用的bhyve虚拟机管理系统，通过它，你只需要简单的输入几行命令就能创建自己的虚拟机，包括win、linux以及各种版本的BSD系统。

## 1、前期准备


```
##1.安装vm-bhyve

pkg install vm-bhyve

##2.为虚拟机创建数据集，我的zpool名称为zroot

zfs create zroot/vm

##3.在 /etc/rc.conf 中启用 vm-bhyve 并设置要使用的数据集

sysrc vm_enable="YES"
sysrc vm_dir="zfs:zroot/vm"
```

## 2、初始化模板和网络

```
##1.运行 `vm init` 命令以在$vm_dir 下创建所需的目录并加载内核模块

vm init

##2.安装 vm-bhyve 附带的示例模板

cp /usr/local/share/examples/vm-bhyve/* /mountpoint/for/pool/vm/.templates/

##3.创建一个名为 'public' 的虚拟交换机，并将您的网络接口连接到它，此处我的网络接口名称为re0

vm switch create public
vm switch add public re0
```

## 3、创建debian虚拟机

```
##1.下载虚拟机需要使用的debian镜像，镜像源建议选择国内源

vm iso https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-12.8.0-amd64-netinst.iso

##2.虚拟机模板修改，我在/zroot/vm/.templates/目录下创建自己的debian-mine.conf模板，以下是模板内容

loader="grub"
cpu=4 #根据自己cpu配置选择
memory=2048M #根据自己的内存数量填写
network0_type="virtio-net"
network0_switch="public" #选择自己新建的虚拟交换机
disk0_type="virtio-blk"
disk0_name="disk0"
grub_run_partition="1"
grub_run_dir="/boot/grub"
uuid="f89d6ce1-a255-11ef-a32b-00e01e49a0b8"
network0_mac="58:9c:fc:0d:cc:29"

##3.创建虚拟机，虚拟机名称为debian

vm create debian

##4.安装系统镜像，使用debian-mine.conf模板

vm install debian-mine debian-12.8.0-amd64-netinst.iso

##5.完成安装

根据debian安装引导，完成安装

```

## 4、安装成功

- 使用vm list可以看到自己安装的虚拟机列表，如下图所示：

<div>
  <img src="/assets/vm-list.png" class="sm:w-1/2 mx-auto" alt="coding dev illustration">
</div>

- 使用vm console debian，输入用户名密码进入虚拟机后台，执行相关操作比如部署docker等服务

<div>
  <img src="/assets/debian-console.png" class="sm:w-1/2 mx-auto" alt="coding dev illustration">
</div>

## 5、在虚拟机安装1panel

使用curl -sSL https://resource.fit2cloud.com/1panel/package/quick_start.sh -o quick_start.sh && bash quick_start.sh安装1panel，并按照指引完成安装。下图为1panel安装完成部署的jellyfin容器。

<div>
  <img src="/assets/1panel.png" class="sm:w-1/2 mx-auto" alt="coding dev illustration">
</div>