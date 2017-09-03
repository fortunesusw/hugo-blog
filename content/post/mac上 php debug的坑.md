---
title: "Mac上 Php Debug的坑"
date: 2017-09-03T18:10:20+08:00
subtitle: ""
tags: ["php", "xdebug"]
---

之前使用nginx + apache， debug相当畅快。 后面为了方便，直接使用nginx + php-fpm。

nginx部署自不必说， 没问题， 但部署完php-fpm， 便不能debug了...

码代码不能debug， 就像脱光不能XX一样， 难受， 特别是对开源的或者其他项目做二次开发。


<!--more-->

在使用`apache`好好的， 切换到`php-fpm`就出问题了。

马上google找解决办法， 找了很多， 试了却不生效， 搞到昨天凌晨3点多， 第二天继续找问题， 终于在刚才解决了， 时间流水啊， 肉疼。

感慨对`php`不熟悉啊， 不像`java`， 随便撸，都能得以解决。


## 问题原因

由于mac有自带的`php php-fpm`，但扩展这些都没有， 由于不了解， 自带的`php-fpm`不能debug， 所以折腾了一天多。

解决过程不堪回首， 各种安装， 各种`ln -s`某些包， 还是没成功， 后面直接把`mysql、php`卸载重新安装， 包括`xdebug`。 竟然OK了， WTF....

其中重新安装的php命令

```
#5.6版本有自带fpm
brew install php56 --with-debug --with-homebrew-curl --with-homebrew-libressl --with-homebrew-libxslt  --with-libmysql --without-apache
```

```
#fpm启动， 千万不能使用/usr/sbin/php-fpm mac上自带的。。。大坑
/usr/local/sbin/php-fpm -c /usr/local/etc/php/5.6/php.ini -y /usr/local/etc/php/5.6/php-fpm.conf -D
```


mac虽然方便， 但坑起来也折腾啊。。。。

