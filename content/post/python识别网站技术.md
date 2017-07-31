---
title: "Python识别网站技术"
date: 2017-07-31T19:10:43+08:00
subtitle: ""
tags: [python]
---

探测网站使用什么语言开发，服务框架等

<!--more-->

# python 识别网站所用技术
> [builtwith](https://pypi.python.org/pypi/builtwith)
> [python3中使用builtwith的方法](http://www.itwendao.com/article/detail/332500.html)


```
>>> builtwith('http://wordpress.com')
{u'blogs': [u'PHP', u'WordPress'], u'font-scripts': [u'Google Font API'], u'web-servers': [u'Nginx'], u'javascript-frameworks': [u'Modernizr'], u'programming-languages': [u'PHP'], u'cms': [u'WordPress']}
>>> builtwith('http://webscraping.com')
{u'javascript-frameworks': [u'jQuery', u'Modernizr'], u'web-frameworks': [u'Twitter Bootstrap'], u'web-servers': [u'Nginx']}
>>> builtwith('http://microsoft.com')
{u'javascript-frameworks': [u'jQuery'], u'mobile-frameworks': [u'jQuery Mobile'], u'operating-systems': [u'Windows Server'], u'web-servers': [u'IIS']}
>>> builtwith('http://jquery.com')
{u'cdn': [u'CloudFlare'], u'web-servers': [u'Nginx'], u'javascript-frameworks': [u'jQuery', u'Modernizr'], u'programming-languages': [u'PHP'], u'cms': [u'WordPress'], u'blogs': [u'PHP', u'WordPress']}
>>> builtwith('http://joomla.org')
{u'font-scripts': [u'Google Font API'], u'miscellaneous': [u'Gravatar'], u'web-servers': [u'LiteSpeed'], u'javascript-frameworks': [u'jQuery'], u'programming-languages': [u'PHP'], u'web-frameworks': [u'Twitter Bootstrap'], u'cms': [u'Joomla'], u'video-players': [u'YouTube']}
```




