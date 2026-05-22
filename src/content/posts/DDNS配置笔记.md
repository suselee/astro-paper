---
author: Suse
pubDatetime: 2024-11-27T15:34:00Z
title: DDNS配置笔记
featured: true
draft: false
tags:
  - linux
  - nas
description: 获取公网地址后配置ddns开启远程访问
---

不知道为什么？今年开始疯狂痴迷组建nas系统，truenas、xigmanas相关的知识看了又看，也一直计划使用什么样的硬件方案，最终定了H610+G6900的方案，正在马上要实施的关头，结果买了一台二手的游戏主机，nas计划被老婆无限后延了。

众所周知ipv4地址数量严重不足，而自组nas最关键的问题就是如何实现远程访问家里的设备！为了实现这一目的，我们不仅要找运营商沟通索要公网ip地址，还要学会如何配置DDNS实现内网设备远程访问。

联通的公网ipv4地址还是比较好获取的，我这边的获取方式是拨号时后面加上“@adsl8”，然后就发现分配给的ip地址为公网地址了。

有了公网地址后就需要准备内网想映射出去的服务了，刚好前几天我的小鸡vps内存太小，又想部署n8n，所以把上面的ttyrss迁移到家里的n1盒子上来了，经过测试在内网访问ttyrss服务一切正常，路由器配置好端口映射。

关于域名，是必须要有的，当时通过那种ddns服务商提供的二级域名也能实现外网访问的需求，但是大多收费，到不如自己注册一个来的方便，有用不完的子域名可用..，我的域名刚注册的，时间10年，妥妥的。

最后就是ddns解析了，其实我刚换的中兴路由器也带ddns功能，但是里面提供服务的厂商大都收费，尝试了一下就放弃了，听说ddns-go非常好用，何不把ddns-go安装到n1盒子上呢，操作的步骤很简单，按照教程下载对应架构的二进制包，因为我的是n1，arm64架构的，所以下载的是该[链接](https://github.com/jeessy2/ddns-go/releases/download/v6.6.5/ddns-go_6.6.5_linux_arm64.tar.gz)。

```
#赋予执行权限
chmod u+x ddns-go
#安装
./ddns-go -s install
```

然后使用浏览器登陆http://盒子IP:9876即可登录进行相关的配置。

==注意登陆管理页面后，最好十分钟之内配置完成==

我使用cloudflare进行解析，主要配置如下：

### 1. 创建API令牌

![api令牌](/assets/api-token.png)

### 2. 编辑区域DNS

![dns](/assets/编辑DNS.png)

### 3. 创建令牌

![创建令牌](/assets/创建令牌.png)
权限选择编辑，然后再添加一个区域，DNS读取。
区域资源选择自己的域名。

### 4. DDNS-GO配置、填写Token

![ddns-go](/assets/ddns-go.png)

### 5. 保存

![ddns-go](/assets/save.png)

### 6. 测试

不出意外，此时cloudflare已经可以正常解析自己的域名，通过该域名也可以正常访问搭建在内网的服务，大功告成！

![success](/assets/success.png)
