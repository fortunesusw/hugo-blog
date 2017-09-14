---
title: "&#X开头的是什么编码?"
date: 2017-09-14T11:28:13+08:00
subtitle: ""
tags: ['unicode']
---

> 首先说明，这不是编码

在使用readability的api提取网页内容的时候 [点这里看-readability-网页内容提取利器](http://www.jianshu.com/p/b9cbb843e807) ，中文内容都是 `&#x` 开头的一堆乱码似的东西。但保存成网页文件后，浏览器是可以正常显示的

<!--more-->

故搜索了一下，知乎上有个 [回答](https://www.zhihu.com/question/21390312/answer/18091465) 挺好，在此转一下：

形如

```
&name;
&#dddd;
&#xhhhh;
```

的一串字符是 HTML、XML 等 SGML 类语言的转义序列（escape sequence）。 **它们不是「编码」** 。以 HTML 为例，这三种转义序列都称作 character reference：第一种是 character entity reference，后接预先定义的 entity 名称，而 entity 声明了自身指代的字符。

后两种是 numeric character reference（NCR），数字取值为目标字符的 Unicode code point；以「&\#」开头的后接十进制数字，以「&\#x」开头的后接十六进制数字。

从 HTML 4 开始，NCR 以 Unicode 为准，与文档编码无关。「中国」二字分别是 Unicode 字符 U+4E2D 和 U+56FD，十六进制表示的 code point 数值「4E2D」和「56FD」就是十进制的「20013」和「22269」。所以

```
我给加了空格，不然网页会自动渲染成文字...
&#x 4e2d;&#x 56fd;
&# 20013;&# 22269;
```

这两种 NCR 写法都会在显示时转换为「中国」二字。NCR 可以用于转义任何 Unicode 字符，而 character entity reference 很受限，参见 HTML 4 和 HTML5 中已有定义的字符列表：
[Character entity references in HTML 4](http://www.w3.org/TR/html401/sgml/entities.html)
[Character entity references in HTML5](http://dev.w3.org/html5/html-author/charref)

另外可以参考这篇文章 [使用&#x 3000等空格实现最小成本中文对齐](http://www.zhangxinxu.com/wordpress/?p=4562)

知道了是什么，现在来看怎么把它转回成中文呢？
Python实现

要将16进制字符转成中文可以用如下方法

```python
# Pythone3
b'\u4e2d\u6587'.decode('unicode-escape') 
# Python2
'\u4e2d\u6587'.decode('unicode-escape')
```

故需要将 `&#xhhhh;` 做替换，再用上面的方式进行转换。对于特殊符号（如加减乘除），会显示为 `&#xhh` ，后面只有两位，在转换之前，需要提前补全。具体可参看 [readability-网页内容提取利器](http://www.jianshu.com/p/b9cbb843e807)

Java实现

```java
//引用此处：http://xuelianbobo.iteye.com/blog/2155114 
package com.xue.tools; 
import java.io.BufferedWriter; 
import java.io.FileWriter; 
import java.io.IOException; 
import java.net.MalformedURLException; 
import java.net.URL; 
import java.util.List; 
import java.util.regex.Matcher; 
import java.util.regex.Pattern; 
import org.dom4j.DocumentException; 
import org.htmlcleaner.HtmlCleaner; 
import org.htmlcleaner.TagNode; 
import org.htmlcleaner.XPatherException; 

public class Test { 
    public static void main(String[] args) throws IOException, DocumentException, XPatherException { 
        // 定义正则表达式来搜索中文字符的转义符号 
        Pattern compile = Pattern.compile("&#.*?;"); 
        // 测试用中文字符 
        String sourceString = "测试&#x"; 
        Matcher matcher = compile.matcher(sourceString); 
        // 循环搜索 并转换 替换 
        while (matcher.find()) { 
            String group = matcher.group(); 
            // 获得16进制的码 
            String hexcode = "0" + group.replaceAll("(&\#|;)", ""); 
            // 字符串形式的16进制码转成int并转成char 并替换到源串中 
            sourceString = sourceString.replaceAll(group, (char) Integer.decode(hexcode).intValue() + ""); 
        } 
        System.out.println(sourceString); 
    } 
}
```

补充：
后来想到，其实可以把这当成html来解析啊，然后就有了：

```
text = '&#x 4e2d'
lxml.html.fromstring(text).text
```

## 参照

[&#x开头的是什么编码?](http://www.jianshu.com/p/6dcefb2a59b2)

