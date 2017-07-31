---
title: python yield关键字
<!-- subtitle:  -->
date: 2017-07-31
tags: ["python"]
---

> python yield关键字总结

<!--more-->

## 概要

1. 通常的`for...in...`循环中，in后面是一个数组，这个数组就是一个可迭代对象或者链表，字符串，文件等。它的缺陷是所有数据都在内存中，如果有海量数据的话将会非常耗内存。
2. 生成器是可以迭代的，但只可以读取它一次。因为用的时候才生成。比如 `mygenerator = (x*x for x in range(3))`，注意这里用到了`()`，它就不是数组。
3. 生成器`(generator)`能够迭代的关键是它有一个`next()`方法，工作原理就是通过重复调用next()方法，直到捕获一个异常。
4. 带有 `yield` 的函数不再是一个普通函数，而是一个生成器`generator`，可用于迭代，工作原理同上。
5. `yield` 是一个类似 `return` 的关键字，迭代一次遇到`yield`时就返回`yield`后面的值。重点是：下一次迭代时，从上一次迭代遇到的yield后面的代码开始执行。
6. 简要理解：`yield`就是 `return` 返回一个值，并且记住这个返回的位置，下次迭代就从这个位置后开始。
7. 带有`yield`的函数不仅仅只用于`for`循环中，而且可用于某个函数的参数，只要这个函数的参数允许迭代参数。比如`array.extend`函数，它的原型是`array.extend(iterable)`。
8. `send(msg)`与`next()`的区别在于`send`可以传递参数给`yield`表达式，这时传递的参数会作为`yield`表达式的值，而`yield`的参数是返回给调用者的值。——换句话说，就是`send`可以强行修改上一个`yield`表达式值。比如函数中有一个`yield`赋值，`a = yield 5`，第一次迭代到这里会返回5，a还没有赋值。第二次迭代时，使用`.send(10)`，那么，就是强行修改`yield 5`表达式的值为10，本来是5的，那么a=10
9. `send(msg)`与`next()`都有返回值，它们的返回值是当前迭代遇到`yield`时，`yield`后面表达式的值，其实就是当前迭代中`yield`后面的参数。
10. 第一次调用时必须先`next()`或`send(None)`，否则会报错，`send`后之所以为`None`是因为这时候没有上一个`yield`(根据第8条)。可以认为，`next()`等同于`send(None)`。

## 例子

### 代码示例一

```python
def yield_test(n):  
    for i in range(n):  
        yield call(i)  
        print("i=",i)  
    #做一些其它的事情      
    print("do something.")      
    print("end.")  

def call(i):  
    return i*2  

#使用for循环  
for i in yield_test(5):  
    print(i,",")
    
'''结果
0 ,  
i= 0  
2 ,  
i= 1  
4 ,  
i= 2  
6 ,  
i= 3  
8 ,  
i= 4  
do something.  
end.
'''
```


