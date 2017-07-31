---
title: "Xpath基本用法"
date: 2017-07-31T19:35:04+08:00
subtitle: ""
tags: ["other", "python"]
---

> `xpath`在解析`html`, `xml`很方便， 例如用爬虫去爬网页， 解析的节点提取数据

<!-more-->


## 节点（node）
在 XPath 中，有七种类型的节点：元素、属性、文本、命名空间、处理指令、注释以及文档（根）节点。XML 文档是被作为节点树来对待的。树的根被称为文档节点或者根节点。

以下面这xml文档为例：

```xml
<?xml version="1.0" encoding="ISO-8859-1"?> 
<class> 
    <student> 
        <name gender="boy">Harry Potter</name> 
        <ID>24</ID> 
    </student>
    <student>
        <name gender="girl">Li Rose<font color=red>（monitor）</font></name> 
        <ID>1</ID>
    </student>
</class>
```

上面例子的节点为：

```xml
<classs> （文档节点/根节点） 
<ID>24</ID> （元素节点） 
gender="boy" （属性节点）
```

节点关系

父：每个元素以及属性都有一个父。例子中<student>的父是<class>;
子：元素节点可有零个、一个或多个子。例子中<class>的子是<student>;
兄弟：拥有相同的父的节点。例子中<name>和<ID>是兄弟;
祖先：某节点的父、父的父，等等。
后代：某节点的子、子的子，等等。

## 基本值（或称原子值，Atomic value）
基本值是无父或无子的节点。
上面例子的基本值为：

```
Harry Potter "boy"
```

## 项目（Item）
项目是基本值或者节点。

**注意，以下表达式当然可以混合使用** 

### 1 nodename

选取此节点的所有子节点。

```python
from scrapy import Selector 
def parse(self, response): 
    selector=Selector(response) 
    content=selector.xpath(class)# 选取 class 元素的所有子节点。
```

### 2 /

从根节点选取。

```python
from scrapy import Selector 
def parse(self, response): 
    selector=Selector(response) 
    ''' 选取根元素 class。 注释：假如路径起始于正斜杠( / )，则此路径始终代表到某元素的绝对路径！ ''' 
    content1=selector.xpath(/class) 
    content2=selector.xpath(/class/student)
```

### 3 //

从匹配选择的当前节点选择文档中的节点，而不考虑它们的位置。

```python
from scrapy import Selector 
def parse(self, response): 
    selector=Selector(response) 
    content=selector.xpath("//ID")# 选取所有ID子元素，而不管它们在文档中的位置。
```

### 4 .

选取当前节点。

### 5 ..

选取当前节点的父节点。

### 6 @

选取属性。

```python
from scrapy import Selector 
def parse(self, response): 
    selector=Selector(response) # 选取所有gender="boy"属性的节点，而不管它们在文档中的位置。 
    content=selector.xpath(//[@gender="boy"])
```

### 7 谓语（Predicates）

```python
/class/student[1] #选取属于 class 子元素的第一个 student 元素。 
/class/student[last()] #选取属于 class 子元素的最后一个 student 元素。 
/class/student[last()-1] #选取属于 class 子元素的倒数第二个 student 元素。 
/class/student[position()<3] #选取属于 class 子元素的前两个 student 元素。 
/class/student[@gender] #选取所有拥有名为 gender 的属性的 student 元素。 
/class/student[@gender="boy"] #选取所有拥有 gender="boy" 属性的 student 元素。 
/class/student[ID<50] #选取 class 元素的所有 student 元素，且其中的 ID 元素的值须小于 50。 
/class/student[ID<50]/name #选取 class 元素中的 student 元素的所有 name 元素，且其中的 ID 元素的值须小于 35。
```

### 8 通配符

```
1. * 匹配任何元素节点 
2. @* 匹配任何属性节点 
3. node() 匹配任何类型的节点
```

### 9 拓展

对于如下的xml文档（参照 [http://www.tuicool.com/articles/iqQFBn](http://www.tuicool.com/articles/iqQFBn) ） 

```html
<div id="test2">美女，<font color="red">你的微信是多少？</font><div>
```

如果使用：
`data = selector.xpath('//div[@id="test2"]/text()').extract()[0]`
只能提取到"美女，"；

如果使用：
`data = selector.xpath('//div[@id="test2"]/font/text()').extract()[0]`
又只能提取到"你的微信是多少？"

到底我们要怎样才能把“美女，你的微信是多少”提取出来？

```python
data = selector.xpath('//div[@id="test2"]') 
info = data.xpath('string(.)').extract()[0]
```




