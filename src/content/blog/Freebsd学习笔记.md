---
author: Suse
pubDatetime: 2024-11-13T15:59:00Z
title: Freebsd学习笔记
featured: true
draft: false
tags:
  - 技术
  - 运维
  - 操作系统
description: Freebsd jail
---


从大学时代就对FreeBSD情有独钟，笔记本上单独跑FreeBSD做桌面用了一年多，当时用的桌面环境是Xfce。

现在情怀依旧，身在信息行业多年，接触过很多种类的Linux系统，Centos、Debian、Ubuntu、SUSE、Fedora、麒麟等，也见证了它们之间的分裂。在这个时间段，家里的路由器一直使用Openbsd，深深为BSD系统家族的优雅和简单所折服。并且，最近一直有自组NAS的打算，关于自组NAS的系统开始计划使用Truenas，但是了解该系统的过程中发现官方的主力开发已经转到了基于Debian的Truenas scale，基于FreeBSD 14的Truenas core不会再进行开发；然后就是同样基于FreeBSD的xigmanas，在虚拟机上测试了之后发现文档很少，web页面不甚美观，然后才有了自己使用FreeBSD部署，不使用任何web页面的想法。

在网上查了很多的资料，使用FreeBSD搭建nas系统，我的计划是使用ZFS，创建几个jail跑像nextcloud等服务，然后建一个linux虚拟机跑一些docker服务。所以本着实用为主的态度，这次我不会太过于关于原理性的东西，实操为主，先将笔记分为三个版块：

	1. 如何使用Bastille创建jail
	2. 如何使用vm-bhyve创建debian虚拟机
	3. ZFS基础

### 1、使用Bastille创建jail

*[Bastille](https://bastillebsd.org/) 是一个开源系统，用于在 FreeBSD 上自动部署和管理容器化应用程序。*

Bastille的Getting Started文档写的真是太棒了！！！

经过几天断断续续的学习，现在基本能够无障碍的建立jail、进行磁盘目录挂载以及销毁容器了，因为自己的vps小鸡上一直用的是docker，所以无形中就会进行对比，简单来说就是docker的生态比jail强大太多了，但是我认为jail是最优雅的！举两个使用Bastille创建jail的例子吧。

- **通过Bastille templates创建jellyfin服务**

Bastille有很多模板可以用，这个东西我觉得和docker-compose很像，类似一键编排，执行一次所有的依赖自动安装完成，开箱即用。

首先修改bastille配置文件/usr/local/etc/bastille/bastille.conf，主要修改两处内容：

	1. 修改bootstrap url（将url修改为国内源，比如清华或中科大）

	## bootstrap urls
	bastille_url_freebsd="https://mirrors.ustc.edu.cn/freebsd//releases/"     
	
	2. ZFS配置

	## ZFS options
	bastille_zfs_enable="YES"       ## default: ""，启用ZFS
	bastille_zfs_zpool="zroot"      ## default: ""，填入zpool名称,通过df等命令查看

	3. 网络配置，网络配置这里Bastille及其灵活，像docker一样也有多种模式，在创建jellyfin时我采用了loopback形式，该模式是Bastille最常见的形式，我感觉和docker的bridge模式有点类似。
		a. 配置新的 `bastille0` 环回接口
		
		sysrc cloned_interfaces+=lo1 
		sysrc ifconfig_lo1_name="bastille0" 
		service netif cloneup
		
		b. 定义新的防火墙规则
		创建pf.conf配置文件（如果etc目录下没有的话），并填入以下内容：
		
		ext_if="vtnet0" 
		set block-policy return 
		scrub in on $ext_if all fragment reassemble 
		set skip on lo table <jails> persist 
		nat on $ext_if from <jails> to any -> ($ext_if:0) 
		rdr-anchor "rdr/*" 
		block in all pass out quick keep state 
		antispoof for $ext_if inet 
		pass in inet proto tcp from any to any port ssh flags S/SA keep    state
		c. 启用并开启pf防火墙
		
		sysrc pf_enable=YES 
		service pf start

	4. 创建容器运行需要发行版系统，我使用的是FreeBSD14.1 release。
	
	bastille bootstrap 14.1-RELEASE update

	5. 创建容器(容器地址使用私有地址)

	bastille create jellyfin 14.1-RELEASE 172.16.1.2/24

	6. 拉取Bastille的jellyfin模板

	`bastille bootstrap https://gitlab.com/bastillebsd-templates/jellyfin`

	7. 应用并启动容器

	`bastille template TARGET bastillebsd-templates/jellyfin`

	
### 2、jail相关截图

<div>
  <img src="/assets/jellfin1.png" class="sm:w-1/2 mx-auto" alt="coding dev illustration">
</div>

<div>
  <img src="/assets/jellfin2.png" class="sm:w-1/2 mx-auto" alt="coding dev illustration">
</div>

<div>
  <img src="/assets/jellfin3.png" class="sm:w-1/2 mx-auto" alt="coding dev illustration">
</div>

### 3、未完待续
