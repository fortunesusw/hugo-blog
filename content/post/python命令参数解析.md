---
title: "Python命令参数解析"
date: 2017-07-31T19:44:07+08:00
subtitle: ""
tags: ["python"]
---

> python中 getopt 模块，该模块是专门用来处理命令行参数的
> 函数`getopt(args, shortopts, longopts = [])`
> 参数args一般是`sys.argv[1:]`，shortopts 短格式 (-)， longopts 长格式(--)


<!--more-->

## 代码部分

命令行中输入：

```python
python3 args.py -h
python3 args.py -f abc -t to --subject khkhj -c ad
```

```python
#!/usr/bin/python3
import getopt
import sys

def usage():
    print('usage:\n -h help\n -f from\n -t to\n -s subject\n -c content\n')
    sys.exit()

if __name__ == '__main__':
    if len(sys.argv) < 2:
        usage()
    try:
        options, args = getopt.getopt(sys.argv[1:], "hf:t:s:c:", ['help', "from=", "to=", "subject=", "content="])
        for name, value in options:
            if name in ('-h', '--help'):
                usage()
            elif name in ('-f', '--from'):
                print(value)
            elif name in ('-t', '--to'):
                print(value)
            elif name in ('-s', '--subject'):
                print(value)
            elif name in ('-c', '--content'):
                print(value)
    except getopt.GetoptError:
        usage()
```

## 参数说明
> options, args = getopt.getopt(sys.argv[1:], "hf:t:s:c:", ['help', "from=", "to=", "subject=", "content="])

```
hp:i: 短格式
h 后面没有冒号：表示后面不带参数，p：和 i：后面有冒号表示后面需要参数

['help', "from=", "to=", "subject=", "content="]长格式
help后面没有等号=，表示后面不带参数，其他三个有=，表示后面需要参数
```

返回值 options 是个包含元祖的列表，每个元祖是分析出来的格式信息，比如 `[(‘-f’, 'abc'), ('-t', 'to'), ('--subject', 'khkhj'), ('-c', 'ad’)]`
args 是个列表，包含那些没有‘-’或‘--’的参数，比如：['55','66']

**注意：定义命令行参数时，要先定义带'-'选项的参数，再定义没有'-'的参数**
