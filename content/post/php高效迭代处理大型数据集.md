---
title: "Php高效迭代处理大型数据集"
date: 2017-08-05T15:12:53+08:00
subtitle: "php实践之生成器"
tags: ["php"]
---

希望高效快速迭代处理一个元素列表， 不过整个列表会占用大量内存， 或者生成整个列表的速度非常慢

<!--more-->

## 解解办法
使用生成器

```php
function FileLineGenerator($file) { 
    if (!$fh = fopen($file, 'r')) {
        return; 
    }
    while (false !== ($line = fgets($fh))) { 
        yield $line;
    }
    fclose($fh);
}
$file = FileLineGenerator('log.txt'); 
foreach ($file as $line) {
    if (preg_match('/^rasmus: /', $line)) { 
        print $line; 
    }
}
```

## 讨论
生成器提供了一个简单的办法， 可以高效地循环处理元素， 而不是将所有数据加载到一个数组中导致巨大开销

生成器是返回一个可迭代对象。 在循环处理对象时， PHP会反复调用生成器来得到下一个值， 这是`yield`关键字作用。

生成器最适合来处理一个文件中的所有行， 最简单的方法是使用`file()`函数但这样会将所有行都加载到内存（数组）中， 然后关闭文件。