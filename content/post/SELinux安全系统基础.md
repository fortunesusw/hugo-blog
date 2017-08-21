---
title: "SELinux安全系统基础"
date: 2017-08-21T14:37:26+08:00
subtitle: ""
tags: ["linux"]
---

操作系统的安全机制其实就是对两样东西做出限制：**进程和系统资源**(文件、网络套接字、系统调用等)。

<!--more-->

## 一、SELinux简介

`SELinux(Secure Enhanced Linux)`安全增强的`Linux`是由美国国家安全局NSA针对计算机基础结构安全开发的一个全新的`Linux`安全策略机制。`SELinux`可以允许系统管理员更加灵活的来定义安全策略。

`SELinux`是一个内核级别的安全机制，从`Linux2.6`内核之后就将SELinux集成在了内核当中，因为SELinux是内核级别的，所以我们对于其配置文件的修改都是需要重新启动操作系统才能生效的。

现在主流发现的Linux版本里面都集成了`SELinux`机制，`CentOS/RHEL`都会默认开启`SELinux`机制。

## 二、SELinux基本概念

Linux操作系统是通过用户和组的概念来对我们的系统资源进行限制，我们知道每个进程都需要一个用户才能执行。

在`SELinux`当中针对这两样东西定义了两个基本概念：`域(domin)`和`上下文(context)`。

**域就是用来对进程进行限制，而上下文就是对系统资源进行限制。**

我们可以通过 `ps -Z`命令来查看当前进程的域的信息，也就是进程的SELinux信息：

```
[root@localhost ~] ps -Z
LABEL                             PID TTY          TIME CMD
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 8473 pts/1 00:00:00 bash
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 17669 pts/1 00:00:00 ps
```

通过 `ls -Z` 命令我们可以查看文件上下文信息，也就是文件的SELinux信息：

```
[root@localhost ~] ls -Z
-rw-r--r--. root root unconfined_u:object_r:admin_home_t:s0 abc
-rw-r--r--. root root unconfined_u:object_r:admin_home_t:s0 access.com
```

```
unconfined_u:object_r:admin_home_t:s0

这条语句通过：划分成了四段，第一段 unconfined_u 代表的是用户，第二段 object_r 表示的是角色，第三段是SELinux中最重要的信息，admin_home 表示的是类型，最后一段 s0 是跟MLS、MCS相关的东西
```

* unconfined_u: 指的是`SElinux`用户，`root`表示`root`账户身份，`user_u`表示普通用户无特权用户，`system_u`表示系统进程，通过用户可以确认身份类型，一般搭配角色使用。身份和不同的角色搭配时有权限不同，虽然可以使用su命令切换用户但对于SElinux的用户并没有发生改变，账户之间切换时此用户身份不变，在targeted策略环境下用户标识没有实质性作用。

* object_r: `object_r`一般为文件目录的角色、`system_r`一般为进程的角色，在targeted策略环境中用户的角色一般为`system_r`。用户的角色类似用户组的概念，不同的角色具有不同的身份权限，一个用户可以具备多个角色，但是同一时间只能使用一个角色。在targeted策略环境下角色没有实质作用，在targeted策略环境中所有的进程文件的角色都是`system_r`角色。

* admin_home: 文件和进程都有一个类型，`SElinux`依据类型的相关组合来限制存取权限。


## 三、策略

在`SELinux`中，我们是通过定义策略来控制哪些域可以访问哪些上下文。

在`SELinux`中，预置了多种的策略模式，我们通常都不需要自己去定义策略，除非是我们自己需要对一些服务或者程序进行保护

在`CentOS/RHEL`中，其默认使用的是目标(target)策略，那么何为目标策略呢？

目标策略定义了只有目标进程受到`SELinux`限制，非目标进程就不会受到SELinux限制，通常我们的网络应用程序都是目标进程，比如`httpd`、`mysqld`，`dhcpd`等等这些网络应用程序。

我们的`CentOS`的`SELinux`配置文件是存放在 `/etc/sysconfig/selinux` 文件，我们可以查看一下里面的内容：

```
[root@localhost ~] cat /etc/sysconfig/selinux

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=enforcing    #SELinux默认的工作模式是enforcing
# SELINUXTYPE= can take one of these two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted  #CentOS使用的策略就是目标策略
```

## 四、 SELinux模式
配置文件`/etc/sysconfig/selinux`

`SELinux`的工作模式一共有三种 `enforcing`、`permissive`和`disabled `

* enforcing(强制模式)：只要是违反策略的行动都会被禁止，并作为内核信息记录
* permissive(允许模式)：违反策略的行动不会被禁止，但是会提示警告信息
* disabled(禁用模式)：禁用`SELinux`，与不带`SELinux`系统是一样的，通常情况下我们在不怎么了解`SELinux`时，将模式设置成`disabled`，这样在访问一些网络应用时就不会出问题了。


我们如果要查看当前`SELinux`的工作状态，可以使用 `getenforce` 命令来查看：

```
[root@localhost ~] getenforce
Enforcing
```

当前的工作模式是 `enforcing`，我们如果要设置当前的`SELinux`工作状态，可以使用 `setenforce [0|1]` 命令来修改，`setenforce 0`表示设置成 `permissive`，1表示`enforcing`

**【注意：】通过 `setenforce` 来设置`SELinux`只是临时修改，当系统重启后就会失效了，所以如果要永久修改，就通过修改`SELinux`主配置文件**

```
[root@localhost ~] getenforce
Permissive
[root@localhost ~] getenforce 1
Permissive
[root@localhost ~] setenforce 1
[root@localhost ~] getenforce
Enforcing
```

## 五、 实例

```
[root@localhost ~] ls -Z -d /var
drwxr-xr-x. root root system_u:object_r:var_t:s0       /var
[root@localhost ~] ls -Z -d /var/www/
drwxrwxrwx. root root system_u:object_r:httpd_sys_content_t:s0 /var/www/
[root@localhost ~] ls -Z -d /var/www/html/
drwxrwxrwx. root root system_u:object_r:httpd_sys_content_t:s0 /var/www/html/
[root@localhost ~] ls -Z -d /var/www/html/plibli/
drwxrwxrwx. apache apache unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/plibli/
```

**一般情况下它们会继承上一级目录的context值，但是，一些安装服务产生的文件context值会例外，不继承上级目录的context值，服务会自动创建它们的context值，比如没有装http服务的时候/var/目录下时没有www目录的，安装httpd服务后该服务会自动创建出所需的目录，并定义与服务相关的目录及文件才context值，它们并不会继承上级目录的context值。**

当访问`plibli`时， 会出现无法写， 因为我们此时的`SELinux`的工作模式是 `enforcing`，所以对于违反策略的行动是被禁止的。

那么我们这个时候应该解决这个问题呢？

通常解决办法由两种：

1. 直接将`SELinux`的工作模式设置成 `disabled`，这样就不会出现策略拦截问题了，但是这样的话我们的系统就没有`SELinux`安全防护了

2. 通过 `restorecon` 或者 `chcon` 命令来修复我们的文件上下文信息

```
restorecon -R -v /var/www/html/plibli　　//-R 表示递归，如果是目录，则该目录下的所有子目录、文件都会得到修复　　

chcon -R -u system_u plibli
```

## 参照 

* [SELinux安全系统基础](http://www.cnblogs.com/xiaoluo501395377/archive/2013/05/26/3100444.html)
* [SELinux](https://wiki.centos.org/zh/HowTos/SELinux)
* [Confined and Unconfined Users
](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Security-Enhanced_Linux/sect-Security-Enhanced_Linux-Targeted_Policy-Confined_and_Unconfined_Users.html)


