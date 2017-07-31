---
title: "使用httrack制作网站镜像"
date: 2017-07-31T20:33:16+08:00
subtitle: ""
tags: ["crawler"]
---

> httrack 是个非常强大网站镜像工具，可以用来备份你的网站，在 Windows、Linux、MacOSX 上都能使用。本文以命令行方式举例，如果你觉得命令行方式不方便，官方还提供了 GUI 工具（WinHTTrack/WebHTTrack）
> 
> 官网地址：[www.httrack.com](www.httrack.com)


<!--more-->

# 安装
在mac下， 直接使用`brew`

```
brew install httrack
```

# 基本用法
用法也很简洁，总共只有三部分，选项、过滤器规则和目标网址。

```
httrack [-选项] [+/-规则] 站点网址
```

例如：

```
httrack "http://www.all.net/abc/index.html" -O "/tmp/www.all.net" "+*.all.net/*" -v
```

## 常用选项
* -w, --mirror

    镜像模式（默认）

* -W, --mirror-wizard

    按照向导方式开启镜像模式。是一种半自动的的镜像模式，以问答方式填控制参数。

* -O, --path

    站点镜像保存的本地路径，用法例如：`-O 站点镜像文件路径[,缓存及日志路径（可选）]`

* --continue

    断点续传。
    
* -P, --proxy

    代理方式。不过貌似只支持http代理，如果无法使用socks代理可以使用[tsocks](https://www.coderxing.com/install-tsocks-on-macos.html)用法如：

    ```
    -P proxy:port or -P user:pass@proxy:port
    ```

* -F, --user-agent

    设置UserAgent，用法例如：
    
    ```
    -F "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.95 Safari/537.36"
    ```
    
* -V, --userdef-cmd

    在每个文件后执行系统命令，很强大，可以用来修改源文件，用法如，$0 为源文件名，例如 `-V "rm \$0`，表示下载后删除文件。

* -s, --robots

    是否遵守 robots 协议，包括 `robots.txt` 文件以及 HTML 文件中的 robots 标签，（0=不遵守，1=sometimes，2=遵守），默认为遵守（值为 2），例如，`-s0` 或 `--robots=0` 表示不能遵守 robots 协议。
    
* -D, --can-go-down

    只能抓取当前目录以及下级目录的文件，比如：

    ```
    httrack -O /tmp/test  https://developers.google.com/speed/pagespeed/module/
    ```
    只能抓取 `/speed/pagespeed/module/` 同级和下级目录中的文件，比如 `/speed/pagespeed/a.css` 在 `/speed/pagespeed/module/` 的上级，所以不会被抓取。

    该选项默认开启。及默认情况下只抓同级和下级目录的文件。

* -U, --can-go-up

    和 `-D` 相反，`-U` 只能抓取同级和上级目录的文件。

* -B, --can-go-up-and-down

    相当于 `-D` 和 `-U` 的组合，上下级目录的文件都可以抓。

* -a, --stay-on-same-address

    仅在当前域名下抓取，比如上面的例子，只会抓取 `developers.google.com` 域名下的文件，而不包括 `fonts.google.com` 下的文件（在没有指定过滤器的情况下，如果有过滤规则匹配了外部域名，也会被抓取到）。
    
* -d, --stay-on-same-domain

    在当前主域名下抓取，比如上面的例子，不仅包括 `developers.google.com` 而且还可能包括 `fonts.google.com`，但不包括 `static.youbute.com` 下的资源。

* -K, --keep-links

    控制链接如何转换，默认是转成相对路径。

* -c, --sockets

    同时开启几个并发进行抓取，默认 8 个。

* -T, --timeout

    抓取单个链接的超时时间，以秒为单位。

* -R, --retries

    抓取单个链接的失败重试次数，默认为 1。
   
* -%L linkfile

    可以通过链接文件方式批量抓取文件，linkfile 文件中每行一个链接地址。

* -x, --replace-external
    把所有未匹配的外部链接都替换成错误页面，这个在制作离线镜像站点时很有用，有些页面会引用外部资源（如 css、js）文件来渲染页面，如果外部文件无法访问，就会让离线页面长时间无法渲染，看到的只会大白页。

httrack 的配置非常丰富，更多选项参考[官方文档](https://www.httrack.com/html/fcguide.html)，有`*` 标记的为默认值

## 过滤器

过滤器规则放在命令行最后，规则很简单，按照 url 的绝对路径进行匹配，包含指令用`+`，排除`-`。比如 `-*` `+*.jpg` 只下载以 `.jpg` 结尾的图片，另如 `+*font.google.com/*"`表示所有匹配 `font.google.com/` 的文件都会下载。

## 例子

镜像站点 `https://developers.google.com/speed/pagespeed/module/` 以便能离线访问，指令如下：

```
httrack -O /tmp/a "+*/_static/*" "+*/styles/*" "+*.css" "+*.ttf" \
    -x https://developers.google.com/speed/pagespeed/module/
```