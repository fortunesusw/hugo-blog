---
title: "Python装饰器@Staticmethod和@Classmethod有什么区别?"
date: 2017-08-17T15:10:03+08:00
subtitle: ""
tags: ["python"]
---

一般来说，要使用某个类的方法，需要先实例化一个对象再调用方法。
而使用@staticmethod或@classmethod，就可以不需要实例化，直接类名.方法名()来调用。
这有利于组织代码，把某些应该属于某个类的函数给放到那个类里去，同时有利于命名空间的整洁。

<!--more-->

## 实例说明

也许一些例子会有帮助：注意`foo`， `class_foo` 和`static_foo`参数的区别:

```python
class A(object):
    def foo(self,x):
        print "executing foo(%s,%s)"%(self,x)

    @classmethod
    def class_foo(cls,x):
        print "executing class_foo(%s,%s)"%(cls,x)

    @staticmethod
    def static_foo(x):
        print "executing static_foo(%s)"%x

a=A()
```

下面是一个对象实体调用方法的常用方式，对象实体a被隐藏的传递给了第一个参数。

```
a.foo(1)
# executing foo(<__main__.A object at 0xb7dbef0c>,1)
```

用`classmethods`装饰，隐藏的传递给第一个参数的是对象实体的类(class A)而不是self。

```
a.class_foo(1)
# executing class_foo(<class '__main__.A'>,1)
```

你也可以用类调用`class_foo`。实际上，如果你把一些方法定义成classmethod，那么实际上你是希望用类来调用这个方法，而不是用这个类的实例来调用这个方法。`A.foo(1)`将会返回一个TypeError错误，`A.class_foo(1)`将会正常运行:

```
A.class_foo(1)
# executing class_foo(<class '__main__.A'>,1)
```

One use people have found for class methods is to create inheritable alternative constructors.

用staticmethods来装饰，不管传递给第一个参数的是self(对象实体)还是cls(类)。它们的表现都一样:

```
a.static_foo(1)
# executing static_foo(1)

A.static_foo('hi')
# executing static_foo(hi)
```

静态方法被用来组织类之间有逻辑关系的函数。

foo只是个函数，但是当你调用a.foo的时候你得到的不仅仅是一个函数，你得到的是一个第一个参数绑定到a的"加强版"函数。foo需要两个参数，而a.foo仅仅需要一个参数。

a绑定了foo。下面可以知道什么叫"绑定"了:

```
print(a.foo)
# <bound method A.foo of <__main__.A object at 0xb7d52f0c>>
```

如果使用a.class_foo，是A绑定到了class_foo而不是a。

```
print(a.class_foo)
# <bound method type.class_foo of <class '__main__.A'>>
```

最后剩下静态方法，说到底它就是一个方法。a.static_foo只是返回一个不带参数绑定的方法。static_foo和a.static_foo只需要一个参数。

```
print(a.static_foo)
# <function static_foo at 0xb7d479cc>
```


## 关键点

既然@staticmethod和@classmethod都可以直接`类名.方法名()`来调用，那他们有什么区别呢
从它们的使用上来看

1. @staticmethod不需要表示自身对象的self和自身类的cls参数，就跟使用函数一样。
2. @classmethod也不需要self参数，但第一个参数需要是表示自身类的cls参数。
3. staticmethod可以当作静态函数，用类名访问； clssmethod绑定的是类，而非实例， 内部用cls访问属性

如果在@staticmethod中要调用到这个类的一些属性方法，只能直接`类名.属性名`或`类名.方法名`。
而@classmethod因为持有cls参数，可以来调用类的属性，类的方法，实例化对象等，避免硬编码。
下面上代码。


```python

class A(object):  
    bar = 1  
    def foo(self):  
        print 'foo'  
 
    @staticmethod  
    def static_foo():  
        print 'static_foo'  
        print A.bar  
 
    @classmethod  
    def class_foo(cls):  
        print 'class_foo'  
        print cls.bar  
        cls().foo()  
  
A.static_foo()  
A.class_foo() 

#输出：
# static_foo
# 1
# class_foo
# 1
# foo

```