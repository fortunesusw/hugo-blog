---
title: "在Linux上使用PHPCMS的踩坑记录"
date: 2017-08-21T14:47:51+08:00
subtitle: "Linux下目录权限设置"
tags: ["php", "linux"]
---

**环境：centos7**

最近在使用phpcms(feifeicms)，内部基于`Thinkphp`。
安装很简单，直接下载最新包扔上去， 但在启动时，报了`目录 [ ./Runtime/ ] 不可写！`。

<!--more-->

## 查找问题及解决办法
看到不可写， 首先想到的是权限问题， 直接`chown -R 777 Runtime`，赋予最高权限， 打开浏览器， 还是`目录 [ ./Runtime/ ] 不可写！`， 在`chown -R xxx:xxx Runtime`，还是不行。

因为是使用apache web服务， 所以也折腾了下apache的权限设置， 也解决不了。

google....继续google....

终于找到关键字眼[攻城记：Thinkphp框架的项目规划总结和踩坑经验](http://www.cnblogs.com/batsing/p/Thinkphp.html)，里面提到权限问题，`Runtime`目录设置`777`权限， 如果仍不能正常生成缓存文件，检查是否硬盘已满。若系统为`centos7`，则要关闭 `selinux` 防火墙。

[selinux安全系统基础](http://sufuq.com/post/selinux%E5%AE%89%E5%85%A8%E7%B3%BB%E7%BB%9F%E5%9F%BA%E7%A1%80/)

`chcon`还是不能解决， 后面直接把`SELinux`的工作模式改为`disabled`， 这个需要重启机器。

折腾了2天多， 没找到合适的解决办法， 直接关掉selinux的安全验证，比较暴力， 不推荐

在此记录下折腾过程， 防止后面遇到类似问题。
