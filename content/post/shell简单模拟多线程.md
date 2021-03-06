---
title: "Shell简单模拟多线程"
date: 2017-07-31T20:39:01+08:00
subtitle: ""
tags: ["shell", "linux"]
---

>使用场景，跑一批数据，对这批数据做检查等，如对一批视频url做下载请求等等


<!--more-->

## 模拟多线程
> 主要是使用split切割文件， 遍历切割的文件做操作

```bash
#!/bin/bash
# sh xxx.sh $input $threads
test $# -lt 2 && echo "arguments error" && exit 1;
input=$1
threads=$2
records="$input.records"
prefix="record-"
isSplit="split"
! test -e $records && cat -n $input > $records
if [ ! -e $isSplit ]; then
    totalCount=`wc -l $input|awk '{print $1}'`
    splitLines=`echo $totalCount/$threads|bc`
    split -l $splitLines $records $prefix
    echo "true" > $isSplit
fi

function process() {
    input=$1
    if [ -e $input.offset ]; then
        offset=`cat $input.offset`
    else
        offset=0
    fi
    cat $input | while read line;
    do
        currentOffset=`echo "$line" | awk '{print $1}'`
        url=`echo "$line" | awk '{print $2}'`
        if [ $currentOffset -ge $offset ]; then
            curl -s -o /dev/null $url  1>&2 2>>"$input.offset.log"
            echo "$currentOffset" > $input.offset
        fi
    done
}

for record in `ls record-*`; do
    if [[ $record != *"offset"* ]]; then
        process $record &
    fi
done
```

## 查看状态

```bash
#!/bin/bash
records=`ls record-* | grep -v offset`
for record in $records; do
    total=`wc -l $record | awk '{print $1}'`
    start=`head -n1 $record|awk '{print $1}'`
    end=`tail -n1 $record |awk '{print $1}'`
    current=`cat "${record}.offset"`
    echo "$record $total $[$current-$start+1] $[$end-$current]"
done
```

## 暂停线程

```bash
ps -ef | grep cat-file | grep -v grep | awk '{print $2}'|xargs -I {} kill -9 {}
```