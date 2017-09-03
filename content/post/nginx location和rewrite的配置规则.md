---
title: "Nginx Location和rewrite的配置规则"
date: 2017-09-03T18:14:40+08:00
subtitle: "location、rewrite配置"
tags: ["nginx"]
---

<!--more-->

## location正则写法

```nginx
location  = / {
  # 精确匹配 / ，主机名后面不能带任何字符串
  [ configuration A ]
}
location  / {
  # 因为所有的地址都以 / 开头，所以这条规则将匹配到所有请求
  # 但是正则和最长字符串会优先匹配
  [ configuration B ]
}
location /documents/ {
  # 匹配任何以 /documents/ 开头的地址，匹配符合以后，还要继续往下搜索
  # 只有后面的正则表达式没有匹配到时，这一条才会采用这一条
  [ configuration C ]
}
location ~ /documents/Abc {
  # 匹配任何以 /documents/Abc 开头的地址，匹配符合以后，还要继续往下搜索
  # 只有后面的正则表达式没有匹配到时，这一条才会采用这一条
  [ configuration CC ]
}
location ^~ /images/ {
  # 匹配任何以 /images/ 开头的地址，匹配符合以后，停止往下搜索正则，采用这一条。
  [ configuration D ]
}
location ~* \.(gif|jpg|jpeg)$ {
  # 匹配所有以 gif,jpg或jpeg 结尾的请求
  # 然而，所有请求 /images/ 下的图片会被 config D 处理，因为 ^~ 到达不了这一条正则
  [ configuration E ]
}
location /images/ {
  # 字符匹配到 /images/，继续往下，会发现 ^~ 存在
  [ configuration F ]
}
location /images/abc {
  # 最长字符匹配到 /images/abc，继续往下，会发现 ^~ 存在
  # F与G的放置顺序是没有关系的
  [ configuration G ]
}
location ~ /images/abc/ {
  # 只有去掉 config D 才有效：先最长匹配 config G 开头的地址，继续往下搜索，匹配到这一条正则，采用
    [ configuration H ]
}
location ~* /js/.*/\.js
```

* 以 `=` 开头表示精确匹配
* 如 `A` 中只匹配根目录结尾的请求，后面不能带任何字符串。
* `^~` 开头表示`uri`以某个常规字符串开头，不是正则匹配
* `~` 开头表示区分大小写的正则匹配;
* `~*` 开头表示不区分大小写的正则匹配
* `/` 通用匹配, 如果没有其它匹配,任何请求都会匹配到

**优先级：**

```
(location =) > (location 完整路径) > (location ^~ 路径) > (location ~,~* 正则顺序) > (location 部分起始路径) > (/)
```

### 例子

```
/ -> config A
精确完全匹配，即使/index.html也匹配不了

/downloads/download.html -> config B
匹配B以后，往下没有任何匹配，采用B

/images/1.gif -> configuration D
匹配到F，往下匹配到D，停止往下

/images/abc/def -> config D
最长匹配到G，往下匹配D，停止往下
你可以看到 任何以/images/开头的都会匹配到D并停止，FG写在这里是没有任何意义的，H是永远轮不到的，这里只是为了说明匹配顺序

/documents/document.html -> config C
匹配到C，往下没有任何匹配，采用C

/documents/1.jpg -> configuration E
匹配到C，往下正则匹配到E

/documents/Abc.jpg -> config CC  
最长匹配到C，往下正则顺序匹配到CC，不会往下到E (为什么不会匹配到E)
```

## Rewrite规则
`rewrite`功能就是，使用`nginx`提供的全局变量或自己设置的变量，结合正则表达式和标志位实现`url`重写以及重定向。`rewrite`只能放在`server{}`,`location{}`,`if{}`中，并且只能对`path`，例如 `http://seanlook.com/a/we/index.php?id=1&u=str` 只对`/a/we/index.php`重写。语法`rewrite regex replacement [flag]`。

如果相对域名或参数字符串起作用，可以使用全局变量匹配，也可以使用`proxy_pass`反向代理。

`rewrite`和`location`功能有点像，都能实现跳转，主要区别在于`rewrite`是在同一域名内更改获取资源的路径，而`location`是对一类路径做控制访问或反向代理，可以`proxy_pass`到其他机器。很多情况下`rewrite`也会写在`location`里，它们的执行顺序是：

1. 执行server块的rewrite指令
2. 执行location匹配
3. 执行选定的location中的rewrite指令

如果其中某步`URI`被重写，则重新循环执行1-3，直到找到真实存在的文件；循环超过10次，则返回`500 Internal Server Error`错误。

### flag标志们

* `last` : 相当于`Apache`的`[L]`标记，表示完成`rewrite`
* `break` : 停止执行当前虚拟主机的后续rewrite指令集
* `redirect` : 返回302临时重定向，地址栏会显示跳转后的地址
* `permanent` : 返回301永久重定向，地址栏会显示跳转后的地址

因为301和302不能简单的只返回状态码，还必须有重定向的URL，这就是return指令无法返回301,302的原因了。

**`last` 与 `break`的区别：**
1. `last`一般写在`server`和`if`中，而`break`一般使用在`location`中
2. `last`不终止重写后的`url`匹配，即新的`url`会再从`server`走一遍匹配流程，而`break`终止重写后的匹配
3. `break`和`last`都能组织继续执行后面的`rewrite`指令

### if指令和全局变量
**if判断指令**

语法为`if(condition){...}`，对给定的条件`condition`进行判断。如果为真，大括号内的`rewrite`指令将被执行，if条件(conditon)可以是如下任何内容：

* 当表达式只是一个变量时，如果值为空或任何以0开头的字符串都会当做false
* 直接比较变量和内容时，使用=或!=
* ~正则表达式匹配，~*不区分大小写的匹配，!~区分大小写的不匹配

```
-f和!-f用来判断是否存在文件
-d和!-d用来判断是否存在目录
-e和!-e用来判断是否存在文件或目录
-x和!-x用来判断文件是否可执行
```

例如：

```nginx
if ($http_user_agent ~ MSIE) {
    rewrite ^(.*)$ /msie/$1 break;
} //如果UA包含"MSIE"，rewrite请求到/msid/目录下
if ($http_cookie ~* "id=([^;]+)(?:;|$)") {
    set $id $1;
 } //如果cookie匹配正则，设置变量$id等于正则引用部分
if ($request_method = POST) {
    return 405;
} //如果提交方法为POST，则返回状态405（Method not allowed）。return不能返回301,302
if ($slow) {
    limit_rate 10k;
} //限速，$slow可以通过 set 指令设置
if (!-f $request_filename){
    break;
    proxy_pass  http://127.0.0.1;
} //如果请求的文件名不存在，则反向代理到localhost 。这里的break也是停止rewrite检查
if ($args ~ post=140){
    rewrite ^ http://example.com/ permanent;
} //如果query string中包含"post=140"，永久重定向到example.com
location ~* \.(gif|jpg|png|swf|flv)$ {
    valid_referers none blocked www.jefflei.com www.leizhenfang.com;
    if ($invalid_referer) {
        return 404;
    } //防盗链
}
```

**全局变量**

下面是可以用作if判断的全局变量

* `$args` ： #这个变量等于请求行中的参数，同`$query_string`
* `$content_length` ： 请求头中的`Content-length`字段。
* `$content_type` ： 请求头中的`Content-Type`字段。
* `$document_root` ： 当前请求在`roo`t指令中指定的值。
* `$host` ： 请求主机头字段，否则为服务器名称。
* `$http_user_agent` ： 客户端`agent`信息
* `$http_cookie` ： 客户端`cookie`信息
* `$limit_rate` ： 这个变量可以限制连接速率。
* `$request_method` ： 客户端请求的动作，通常为`GET`或`POST`。
* `$remote_addr` ： 客户端的IP地址。
* `$remote_port` ： 客户端的端口。
* `$remote_user` ： 已经经过`Auth Basic Module`验证的用户名。
* `$request_filename` ： 当前请求的文件路径，由`root`或`alias`指令与`URI`请求生成。
* `$scheme` ： `HTTP`方法（如`http`，`https`）。
* `$server_protocol` ： 请求使用的协议，通常是`HTTP/1.0`或`HTTP/1.1`。
* `$server_addr` ： 服务器地址，在完成一次系统调用后可以确定这个值。
* `$server_name` ： 服务器名称。
* `$server_port` ： 请求到达服务器的端口号。
* `$request_uri` ： 包含请求参数的原始`URI`，不包含主机名，如：`/foo/bar.php?arg=baz`。
* `$uri` ： 不带请求参数的当前URI，$uri不包含主机名，如”/foo/bar.html”。
* `$document_uri` ： 与$uri相同。


## 参考

[nginx配置location总结及rewrite规则写法](http://seanlook.com/2015/05/17/nginx-location-rewrite/)

