---
title: "Redis数据类型及操作"
date: 2017-12-12T15:29:12+08:00
subtitle: ""
tags: ['redis']
---

<!--more-->

> * string
> * hashes
> * lists
> * sets
> * sorted sets
> * Bitmaps
> * HyperLogLogs

## strings类型及操作

`string`是最简单的类型，是二进制安全的。意思是`redis`的`string`可以包含任何数据，比如 `jpg` 图片或者序列化的对象。从内部实现来看其实 `string` 可以看作 `byte` 数组，最大上限是 `1G` 字节

```c
//string类型定义
struct sdshdr {
    long len;
    long free;
    char buf[];
};
```

```

// 相关操作： set get setnx setex setrange mset msetnx get getrange 
// mget incr incrby decr decrby append strlen 等

127.0.0.1:6379> set key1 value1
OK
127.0.0.1:6379> get key1
"value1"
127.0.0.1:6379> setnx key2 value2
(integer) 1
127.0.0.1:6379> get key2
"value2"
```

## hashes类型及操作

`Redis hash`是一个`string`类型的`field`和`value`的映射表.它的添加、删除操作都是`O(1)`(平均)。**`hash` 特别适合用于存储对象**。相较于将对象的每个字段存成单个 `string` 类型。将一个对象存储在 `hash` 类型中会占用更少的内存，并且可以更方便的存取整个对象。省内存的原因是新建一个 `hash` 对象时开始是用 `zipmap`(又称为 `small hash`)来存储的

```

// 相关操作：hset hsetnx hmset hincrby hexists hlen hdel hkeys hvals hgetall

127.0.0.1:6379> hset myhash field1 value1
(integer) 1
127.0.0.1:6379> hset myhash field2 value2
(integer) 1
127.0.0.1:6379> hget myhash field1
"value1"
127.0.0.1:6379> hget myhash field2
"value2"
```

## lists类型及操作

`list` 是一个每个子元素都是 `string` 类型的双向链表，链表的最大长度是`(2 的 32 次方)`，主要功能是 `push`、`pop`、获取一个范围的所有值等等，操作中 `key` 理解为链表的名字。

```
//相关操作： lpush rpush linsert lset lset lrem ltrim lpop rpop rpoplpush lindex llen

127.0.0.1:6379> lpush mylist value1
(integer) 1
127.0.0.1:6379> lpush mylist value2
(integer) 2
127.0.0.1:6379> lrange mylist 0 -1
1) "value2"
2) "value1"
127.0.0.1:6379> rpush mylist2 value1
(integer) 1
127.0.0.1:6379> rpush mylist2 value2
(integer) 2
127.0.0.1:6379> lrange mylist2 0 -1
1) "value1"
2) "value2"
```

## sets类型及操作
`set` 是 `string` 类型的无序集合，对集合的操作有添加删除元素，有对多个集合
求交并差等操作，操作中 `key` 理解为集合的名字。

`set` 的是通过 `hash table` 实现的，所以添加、删除和查找的复杂度都是 `O(1)`

```
// 相关操作： sadd smembers srem spop sdiff sdiffstore sinter sinterstore
// sunion sunionstore smove scard sismember srandmember

127.0.0.1:6379> sadd myset 'hello'
(integer) 1
127.0.0.1:6379> sadd myset 'world'
(integer) 1
127.0.0.1:6379> sadd myset 'world'
(integer) 0
127.0.0.1:6379> smembers myset
1) "hello"
2) "world"
```

## sorted sets 类型及操作
在 `set` 的基础上增加了一个顺序属性，这一属性在添加修改元素的时候可以指定，每次指定后，`zset` 会自动重新按新的值调整顺序

```
// 相关操作：zadd zrange zrem zincrby zrank zrevrank zrevrange zrangebyscore zcount zcard zscore zremrangebyrank zremrangebyscore

127.0.0.1:6379> zadd myzset 1 'one'
(integer) 1
127.0.0.1:6379> zadd myzset 3 'three'
(integer) 1
127.0.0.1:6379> zrange myzset 0 -1
1) "one"
2) "three"
127.0.0.1:6379> zadd myzset 2 'three'
(integer) 0
127.0.0.1:6379> zrange myzset 0 -1
1) "one"
2) "three"
127.0.0.1:6379> zrange myzset 0 -1 withscores
1) "one"
2) "1"
3) "three"
4) "2"
```

## redis 常用命令

命令|说明
---|---
keys| 返回满足给定 `pattern` 的所有 `key`
exists|确认一个 `key` 是否存在
del|删除一个`key`
expire|设置过期时间（单位： 秒）
ttl| 查看一个`key`的剩余过期时间
move|将当前数据库中的`key`转移到其它数据库中
persist|移除给定 `key` 的过期时间
randomkey|随机返回一个 `key`
rename|重命名`key`
type| 返回值的类型
ping|测试连接
echo|打印
select|选择数据库，`Redis` 数据库编号从 `0~15`
dbsize|返回当前数据库的key数目
info|服务器的信息和统计
monitor|实时转储收到的请求
config get|获取服务器配置信息
flushdb|清空当前数据库所有key
flushall|清空所有数据库的key
