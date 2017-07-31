---
title: "科学上网之Shadowsocks安装及优化加速"
date: 2017-07-31T20:25:33+08:00
subtitle: ""
tags: ["shadowsocks"]
---

<!--more-->

[TOC]

## 一、 服务端安装

官方推荐 Ubuntu 14.04 LTS 作为服务器以便使用 TCP Fast Open。服务器端的安装非常简单。

Debian / Ubuntu:

```bash
apt-get install python-pip
```

CentOS:

```bash
yum install python-setuptools && easy_install pip
```

然后直接在后台运行：

```bash
ssserver -p 8000 -k password -m rc4-md5 -d start
```

当然也可以使用配置文件进行配置，方法创建 `/etc/shadowsocks.json` 文件，填入如下内容：

```json
{
    "server":"ip",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"xxxxx",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": true
}
```

然后使用配置文件在后台运行：

```bash
ssserver -c /etc/shadowsocks.json -d start
```

如果要停止运行，将命令中的 `start` 改成 `stop` 。

**TIPS:** 加密方式推荐使用 `rc4-md5` ，因为 RC4 比 AES 速度快好几倍，如果用在路由器上会带来显著性能提升。旧的 RC4 加密之所以不安全是因为 Shadowsocks 在每个连接上重复使用 key，没有使用 IV。现在已经重新正确实现，可以放心使用。更多可以看 [issue](https://github.com/clowwindy/shadowsocks/issues/178) 。

## 二、 客户端安装
[Ports and Clients · shadowsocks/shadowsocks Wiki · GitHub](https://github.com/shadowsocks/shadowsocks/wiki/Ports-and-Clients)

## 三、 加速优化
下面介绍几种简单的优化方法，也是比较推荐的几种，能够得到立竿见影的效果。当然还有一些黑科技我没提到，如有大神路过，也可留言指出。

### 3.1 内核参数优化

首先，将 Linux 内核升级到 3.5 或以上。

#### 第一步，增加系统文件描述符的最大限数

编辑文件 `limits.conf`

```bash
vim /etc/security/limits.conf
```

增加以下两行

```bash
* soft nofile 51200
* hard nofile 51200
```

启动shadowsocks服务器之前，设置以下参数

```bash
ulimit -n 51200
```

#### 第二步，调整内核参数
修改配置文件 `/etc/sysctl.conf`

```bash
fs.file-max = 51200
net.core.rmem_max = 67108864
net.core.wmem_max = 67108864
net.core.netdev_max_backlog = 250000
net.core.somaxconn = 4096
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 0
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.ip_local_port_range = 10000 65000
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_rmem = 4096 87380 67108864
net.ipv4.tcp_wmem = 4096 65536 67108864
net.ipv4.tcp_mtu_probing = 1
net.ipv4.tcp_congestion_control = hybla
```

修改后执行 `sysctl -p` 使配置生效

### 3.2 锐速

锐速是一款非常不错的TCP底层加速软件，可以非常方便快速地完成服务器网络的优化，配合 ShadowSocks 效果奇佳。目前锐速官方也出了永久免费版本，适用带宽20M、3000加速连接，个人使用是足够了。如果需要，先要在锐速官网注册个账户。

然后确定自己的内核是否在锐速的支持列表里，如果不在，请先更换内核，如果不确定，请使用 [手动安装](http://my.serverspeeder.com/w.do?m=lslm) 。

确定自己的内核版本在支持列表里，就可以使用以下命令快速安装了。

```bash
wget http://my.serverspeeder.com/d/ls/serverSpeederInstaller.tar.gz
tar xzvf serverSpeederInstaller.tar.gzbash serverSpeederInstaller.sh
```

输入在官网注册的账号密码进行安装，参数设置直接回车默认即可，
最后两项输入 y 开机自动启动锐速，y 立刻启动锐速。之后可以通过 `lsmod` 查看是否有appex模块在运行。

到这里还没结束，我们还要修改锐速的3个参数， `vim /serverspeeder/etc/config`

```bash
rsc="1" #RSC网卡驱动模式  
advinacc="1" #流量方向加速  
maxmode="1" #最大传输模式

```

digitalocean vps的网卡支持rsc和gso高级算法，所以可以开启 `rsc="1"` ， `gso="1"` 。

重新启动锐速

```bash
service serverSpeeder restart
```

## 四、 防火墙设置
```bash
iptables -F
iptables -A INPUT -p tcp -m tcp --dport 8388 -j ACCEPT
```

## 五、 参照
[科学上网之 Shadowsocks 安装及优化加速](http://wuchong.me/blog/2015/02/02/shadowsocks-install-and-optimize/)
