---
title: "使用OQL（object"
date: 2017-07-31T20:55:50+08:00
draft: true
tags: ["java"]
---

> OQL(object query language)是一个类SQL来查询java堆信息。 
> 
> OQL允许从java堆过滤/选择想要的信息。虽然VisualVM已经支持诸如`show all instances of class X`的预定义查询，但OQL增加了更多的灵活性。
> 
> OQL基于JavaScript表达式语言。

OQL查询格式如下：

```
select <JavaScript expression to select>
[ from [instanceof] <class name> <identifier>
[ where <JavaScript boolean expression to filter> ] ]
```
from和where子名是可选的，其中类名是完全限定的Java类名(`java.net.URL`)或数组类名`char[]([C)`是char数组名称，`java.io.File[]([java.io.File)`是`java.io.File[]`的名称等等。不过要注意，完全限定类名并不总是在运行时唯一标识Java类。可能有不止一个具有相同名称但由不同加载器加载的Java类。因此，类名允许用类对象的id字符串标识。如果使用instanceof关键字，则选择子类型对象，如果未指定此关键字，则仅选择指定的精确类的实例。

java堆对象被包装成脚本对象以便可以以自然语法访问。例如，java的字段可以用`obj.file_name`来访问，数组可以用`array[index]`。选择的每个Java对象都绑定到from子句中指定的标识符名称的JavaScript变量。

<!--more-->


# 一、 OQL 例子

### 选择所有字符串长度大于等于100
```
select s from java.lang.String s where s.count >= 100
```

### 选择所有int数组元素数量大于等于256
```
 select a from int[] a where a.length >= 256
```

### 打印与正则匹配的内容
```
 select {instance: s, content: s.toString()} from java.lang.String s
    where /java/.test(s.toString())
```

### 打印所有文件的路径
```
select file.path.toString() from java.io.File file
```

### 打印所有类加载器的类史
```
 select classof(cl).name 
    from instanceof java.lang.ClassLoader cl

```

### 显示由给定的id标识的类的实例
```
 select o from instanceof 0xd404b198 o
```

# 二、 OQL内置对象及函数

## 2.1 heap对象
内置的`heap`支持如下方法

### heap.forEachClass 
为每个Java类调用回调函数

```
heap.forEachClass(callback)
```

### heap.forEachObject
为每个Java对象调用回调函数

```
 heap.forEachObject(callback, clazz, includeSubtypes)
```
clazz是选择实例的类，如果没有指定，默认是`java.lang.Object`。includeSubtypes是一个布尔标志，指定是否包含子类型实例， 默认为`true`。

### heap.findClass
找到指定名称的Java类

```
heap.findClass(className)
```
返回的Class对象有如下的属性：

* name: 类名
* superclass: 父类
* statics: 表态变量的键值对
* fields: 字段对象数组，字段对象具有名称，签名属性
* loader: 类加载器

返回的Class对象有如下的方法：

* isSubclassOf: 测试是否给定类是此类的直接或间接子类
* isSuperclassOf: 测试是否给定类是此类的直接或间接超类
* subclasses: 返回直接和间接子类的数组
* superclasses: 返回直接和间接超类的数组

### heap.findObject
找到指定id的对象

```
heap.findObject(stringIdOfObject)
```

### heap.classes
返回所有Java类的枚举

### heap.objects
返回所有对象的枚举

```
heap.objects(clazz, [includeSubtypes], [filter])
```
clazz是选择实例的类，如果没有指定，默认是`java.lang.Object`。includeSubtypes是一个布尔标志，指定是否包含子类型实例， 默认为`true`，此方法接受可选的过滤器表达式来过滤对象的结果集

### heap.finalizables
返回将被回收的对象枚举

### heap.livepaths
返回一个给定对象存在的路径数组。此方法接受可选的第二个参数，它是一个布尔标志。该标志指示是否包含具有弱引用的路径。默认情况下，不包含弱引用的路径。

```
select heap.livepaths(s) from java.lang.String s
```

该数组本身的每个元素都是另一个数组。元素的数组包含路径"引用链"中的对象。

### heap.roots
返回堆的根的枚举

每个Root对象具有如下属性:

* id: 根引用的对象的字符串ID
* type: Root的描述类型（JNI Global，JNI Local，Java Static等）
* description: 根的字符串描述
* referrer: 负责此根或null的线程对象或类对象


## 2.2 heap对象例子

### 访问`java.lang.System`的表态字段`props`

```
  select heap.findClass("java.lang.System").statics.props
  select heap.findClass("java.lang.System").props
```

### 获取`java.lang.String`类的字段数

```
select heap.findClass("java.lang.String").fields.length
```

### 找到给定ID的对象

```
select heap.findObject("0xf3800b58")
```

### 查找所有匹配正则`java.net.*`的类

```
select filter(heap.classes(), "/java.net./.test(it.name)")
```

## 2.3 对象函数(functions on individual objects)

### allocTrace(jobject)
如果可用，将返回给定Java对象的分配站点跟踪，allocTrace返回帧对象的数组
每个frame对象具有以下属性:

* className - 在frame中运行的类名称
* methodName - 在frame中运行的方法名称
* methodSignature - 在frame中运行的方法签名
* sourceFileName - 在frame中运行的类源文件的名称
* lineNumber - 方法中的源行号

### classof(jobject)
返回给定对象的Class对象

返回的Class对象有如下的属性：

* name: 类名
* superclass: 父类
* statics: 表态变量的键值对
* fields: 字段对象数组，字段对象具有名称，签名属性
* loader: 类加载器

返回的Class对象有如下的方法：

* isSubclassOf: 测试是否给定类是此类的直接或间接子类
* isSuperclassOf: 测试是否给定类是此类的直接或间接超类
* subclasses: 返回直接和间接子类的数组
* superclasses: 返回直接和间接超类的数组

```
显示每个引用类型对象的类名
select classof(o).name from instanceof java.lang.ref.Reference o

显示java.io.InputStream的子类
select heap.findClass("java.io.InputStream").subclasses()

显示java.io.BufferedInputStream的超类
select heap.findClass("java.io.BufferedInputStream").superclasses()
```

### forEachReferrer(callback, jobject)
为给定对象的每个引用者调用回调函数

### identical(o1, o2)
返回两个给定的Java对象是否相同

```
select identical(heap.findClass("Foo").statics.bar, heap.findClass("AnotherClass").statics.bar)
```

### objectid(jobject)
返回给定对象的id，该id可以传递给`heap.findObject`，也可以用于对象的比较

```
select objectid(o) from java.lang.Object o
```

### reachables(jobject, excludedFields)
返回从给定的对象传递的对象数组。第二个参数可选，逗号分隔的字段名称被排除在可达性计算之外。字段以class_name.field_name模式编写

```
从每个Properties实例打印所有可达到的对象
select reachables(p) from java.util.Properties p

打印每个java.net.URL中的所有可访问性，但忽略java.net.URL.handler
select reachables(u, 'java.net.URL.handler') from java.net.URL u
```

### referrers(jobject)
返回引用指定对象的所有对象

```
打印每个java.lang.Object实例的被引用数
select count(referrers(o)) from java.lang.Object o

打印指个java.io.File对象的被引用数
select referrers(f) from java.io.File f

打印URL对象被引用数大于等于2
select u from java.net.URL u where count(referrers(u)) > 2
```

### referees(jobject)
返回指定对象直接引用的对象数组

```
打印java.io.File类的所有对象
select referees(heap.findClass("java.io.File"))
```

### refers(jobject)
返回第一个对象是否引用第二个对象

### root(jobject)
如果给定对象是对象的根集合的成员，则此函数将返回为什么是这样的描述性Root对象。
如果给定的对象不是根，那么这个函数返回null

### sizeof(jobject)
以字节为单位返回指定对象的大小

```
select sizeof(o) from int[] o
```

### rsizeof(jobject)
返回给定对象的保留集合的大小（以字节为单位），注意，在堆转储中首次使用此功能可能需要大量时间

```
select rsizeof(o) from instanceof java.lang.HashMap o
```

### toHtml(obj)

```
select "<b>" + toHtml(o) + "</b>" from java.lang.Object o
```

## 2.4 查询多值
可以使用JavaScript对象或数组来选择多个值

```
显示每个线程对象的名称和线程
select { name: t.name? t.name.toString() : "null", thread: t } 
from instanceof java.lang.Thread t
```

## 2.5 数组/迭代器/枚举操作功能

### concat(array1/enumeration1, array2/enumeration2)
连接两个数组或枚举（即返回复合枚举）

### contains(array/enumeration, expression)
返回给定的数组/枚举是否包含表达式的元素

```
内置变量：
it -> currently visited element
index -> index of the current element
array -> array/enumeration that is being iterated
```

```
选择由某些静态字段引用的所有属性对象某些类。
select p from java.util.Properties p
    where contains(referrers(p), "classof(it).name == 'java.lang.Class'")
```

### count(array/enumeration, expression)


```
内置变量：
it -> currently visited element
index -> index of the current element
array -> array/enumeration that is being iterated
```

```
统计匹配正则的类数量
select count(heap.classes(), "/java.io./.test(it.name)")
```

### filter(array/enumeration, expression)

```
内置变量：
it -> currently visited element
index -> index of the current element
array -> array/enumeration that is being iterated
result -> result array/enumeration
```

```
显示所有匹配正则的类
select filter(heap.classes(), "/java.io./.test(it.name)")

显示所有引用URL对象但过滤匹配正则的对象
select filter(referrers(u), "! /java.net./.test(classof(it).name)") from java.net.URL u
```

### length(array/enumeration)
返回元素数量

### map(array/enumeration, expression)

```
内置变量：
it -> currently visited element
index -> index of the current element
array -> array/enumeration that is being iterated
result -> result array/enumeration
```

```
显示java.io.File所有静态字段的名称和值
select map(heap.findClass("java.io.File").statics, "index + '=' + toHtml(it)")
```

### max(array/enumeration, [expression])

```
内置变量：
lhs -> left side element for comparison
rhs -> right side element for comparison
```

```
找到具有最大长度的字符串实例
select max(heap.objects('java.lang.String'), 'lhs.count > rhs.count')
```

### min(array/enumeration, [expression])
参照 max

### sort(array/enumeration, [expression])

```
内置变量：
lhs -> left side element for comparison
rhs -> right side element for comparison
```

```
print all char[] objects in the order of size.
select sort(heap.objects('char[]'), 'sizeof(lhs) - sizeof(rhs)')

print all char[] objects in the order of size but print size as well.
select map(sort(heap.objects('char[]'), 'sizeof(lhs) - sizeof(rhs)'), '{ size: sizeof(it), obj: it }')
```

### top(array/enumeration, [expression], [top])

```
内置变量：
lhs -> left side element for comparison
rhs -> right side element for comparison
```

```
print 5 longest strings
select top(heap.objects('java.lang.String'), 'rhs.count - lhs.count', 5)

print 5 longest strings but print size as well.
select map(top(heap.objects('java.lang.String'), 'rhs.count - lhs.count', 5), '{ length: it.count, obj: it }')
```

### sum(array/enumeration, [expression])
```
return sum of sizes of the reachable objects from each Properties object
select sum(map(reachables(p), 'sizeof(it)')) 
from java.util.Properties p

// or omit the map as in ...
select sum(reachables(p), 'sizeof(it)') 
from java.util.Properties p
```

### toArray(array/enumeration)
This function returns an array that contains elements of the input array/enumeration.

### unique(array/enumeration, [expression])

```
// number of unique char[] instances referenced from any String
select count(unique(map(heap.objects('java.lang.String'), 'it.value')))

// total number of Strings
select count(heap.objects('java.lang.String'))
```

## 2.6 更复杂的例子

### Print histogram of each class loader and number of classes loaded by it
```
 select map(sort(map(heap.objects('java.lang.ClassLoader'), 
   '{ loader: it, count: it.classes.elementCount }'), 'rhs.count - lhs.count'),
   'toHtml(it) + "<br>"')
```
上述查询使用的是，java.lang.ClassLoader有一个称为类java.util.Vector的类的私有字段，Vector有一个名为elementCount的私有字段，它是向量中的元素数。我们使用JavaScript对象和map函数来选择多个值（加载器，计数）。我们使用比较表达式的sort函数对count的结果进行排序（即，加载的类的数量）。


### Show parent-child chain for each class loader instance
```
 select map(heap.objects('java.lang.ClassLoader'),
      function (it) {
         var res = '';
         while (it != null) {
            res += toHtml(it) + "->";
            it = it.parent;
         }
         res += "null";
         return res + "<br>";
      })
```


### Printing value of all System properties
```
select map(filter(heap.findClass('java.lang.System').props.table, 'it != null && it.key != null && it.value != null'),
            function (it) {
                var res = it.key.toString() + ' = ' + it.value.toString();
                return res;
            });
```


# 参考文档
官方文档: [https://visualvm.java.net/oqlhelp.html](https://visualvm.java.net/oqlhelp.html)

