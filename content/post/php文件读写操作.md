---
title: "PHP文件读写操作"
date: 2017-08-03T21:45:02+08:00
subtitle: ""
tags: ["php"]
---

php文件读写方式多种， 这边做个记录， 方便查询

<!--more-->


# 读取操作
> 读取操作的函数有：
> 
> * fread()
> * fgets()
> * fgetc()
> * file_get_contents()
> * file()

## fread
> `fread()`函数用于读取文件（可安全用于二进制文件）。

语法：

```php
string fread( int handle, int length )
```

`fread()`从文件指针 `handle` 读取最多 `length` 个字节。当遇到下列任何一种情况时，会停止读取文件：

* 在读取完最多 `length` 个字节数时
* 达到文件末尾的时候（EOF）
* 对于网络流，当一个包可用时 或（在打开用户空间流之后）已读取了 8192 个字节时

例子： 
> 从文件中读取 10 个字节（包括空格）：

```php
<?php
$filename = "test.txt";
$fh = fopen($filename, "r");
echo fread($fh, "10");
fclose($fh);
?>
```

**提示:如果只是想将一个文件的内容读入到一个字符串中，应该用性能更好的`file_get_contents`**

## fgets
> fgets() 函数用于从文件中读取一行数据，并将文件指针指向下一行。

语法：

```php
string fgets( int handle [, int length] )
```

`fgets()` 从 `handle` 指向的文件中读取一行并返回长度最多为 `length-1` 字节的字符串。碰到换行符（包括在返回值中）、`EOF` 或者已经读取了 `length-1` 字节后停止。如果没有指定 `length` ，则默认为 `1K` ，或者说 `1024` 字节。

例子：

```php
<?php
$fh = @fopen("test.txt","r") or die("打开 test.txt 文件出错！");
// if条件避免无效指针
if($fh){
    while(!feof($fh)) {
        echo fgets($fh), '<br />';
    }
}
fclose($fh);
?>
```

## fgetc
> `fgetc()` 函数用于逐字读取文件数据，直到文件结束。

语法：

```php
string fgetc( resource handle )
```

例子：

```php
<?php
$fh = @fopen("test.txt","r") or die("打开 test.txt 文件出错！");
if($fh){
    while(!feof($fh)) {
        echo fgetc($fh);
    }
}
fclose($fh);
?>
```

## file_get_contents
> `file_get_contents()` 函数用于把整个文件读入一个字符串，成功返回一个字符串，失败则返回 FALSE。

语法：

```php
string file_get_contents( string filename [, int offset [, int maxlen]] )
```

参数说明：

参数|说明
---|---
filename|要读取的文件名称
offset|可选，指定读取开始的位置，默认为文件开始位置
maxlen|可选，指定读取文件的长度，单位字节

例子：

```php
<?php
// 读取同时将换行符转换成 <br />
echo nl2br(file_get_contents('test.txt'));
?>
```

## file
> `file()` 函数用于把整个文件读入一个数组中，数组中的每个单元都是文件中相应的一行，包括换行符在内。成功返回一个数组，失败则返回 FALSE。

语法：

```php
array file( string filename )
```

例子：

```php
<?php
$lines = file('test.txt');
// 在数组中循环并加上行号
foreach ($lines as $line_num => $line) {
    echo "Line #{$line_num} : ",$line,'<br />';
}
?>
```

# 文件写入
> 操作方式与读取类似 写入操作的函数有：
> 
> * fwrite()
> * fputs(fwrite别名)
> * file_put_contents()





