---
title: "Php响应xml的踩坑经历"
date: 2017-09-06T22:58:37+08:00
subtitle: ""
tags: ["php"]
---

使用`ajax`的`XMLRequest`请求`php`， `php`响应`xml`。

但是...  响应结果有些非预期， 浏览器显示：

```
XML declration allowed only at the start of the document
```

<!--more-->

## 踩坑背景

`iframe`从使用到放弃。

有个需求是实现一些功能， 为了方便，直接使用`iframe`一个别人的页面。但是最后放弃了`iframe`。过程如下：

1. 首先使用`iframe`嵌入别人页面， 然后直接使用`jquery`操作，去掉一些广告等，但跨域问题会阻止你操作， 不得以放弃`iframe`
2. 换个思路， 直接把页面源码拿过来使用。 但很多内容是直接通过`js`异步加载的， 同样存在跨域问题， `XMLHttpRequest`不能修改`Referer`，不得不放弃现有的`js`请求操作。（源站有验证`Refer`）
3. 最后抓包分析请求， 自己实现`PHP`代理， `js`的异步请求先到代理， 代理在请求源站。
4. 处理编码问题（`GBK`与`UTF-8`）及`UTF-8`是否带`BOM`

但由于源站有些是响应`xml`， `php`响应`xml`的时候出错了。

![](/images/xml-error.jpg)


## 问题跟踪及分析

首先习惯性一顿`google`， 确保代码实现没问题， 同时抓包查看响应。
这两个都确认没问题， 对比代理请求与源站请求（使用`postman`）， 发现个奇怪问题， 代理请求的响应会多个空行。

![](/images/xml-chrome.jpg)


继续`google`， 找到对应的解决办法：

1. 确保php的闭合标签(`?>`)后面没有多余的空行， 这个是原因
2. 如果无效， 加上`ob_clean();`清除`buffer`

## 其他操作
> 处理编码问题（`GBK`与`UTF-8`）及`UTF-8`是否带`BOM`

### `UTF-8`转`GB18030`

```php
iconv('GB18030', 'UTF-8', $response);
```

### `UTF-8`去除`BOM`

```php
function remove_utf8_bom($text) {
    $bom = pack('H*','EFBBBF');
    $text = preg_replace("/^$bom/", '', $text);
    return $text;
}
```

## 参考

* [php闭合标签输出多余空行使xml页面显示错误的处理](https://segmentfault.com/a/1190000007524785)

* [php中include require utf-8文件时顶部产生空行的](http://ijianbian.com/home/post/detail?id=6093925)

* [php: empty line from nowhere](https://stackoverflow.com/questions/10096142/php-empty-line-from-nowhere)

* [GB2312、GBK、GB18030 这几种字符集的主要区别是什么？](https://www.zhihu.com/question/19677619)
