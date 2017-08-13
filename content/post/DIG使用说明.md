---
title: "DIG使用说明"
date: 2017-08-13T12:50:17+08:00
subtitle: ""
tags: ["dns"]
---

Dig(Domain Information Groper)域名信息查询工具是一个在类Unix命令行模式下查询DNS包括NS记录，A记录，MX记录等相关信息的工具

<!--more-->

## 查询DNS

运行命令：

```
dig baidu.com
```

输出为：

```
; <<>> DiG 9.9.5-3-Ubuntu <<>> baidu.com

;; global options: +cmd

;; Got answer:

;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 7794

;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 5, ADDITIONAL: 6



;; OPT PSEUDOSECTION:

; EDNS: version: 0, flags:; udp: 4096

;; QUESTION SECTION:

;baidu.com. IN A



;; ANSWER SECTION:

baidu.com. 76 IN A 123.125.114.144

baidu.com. 76 IN A 220.181.111.85

baidu.com. 76 IN A 220.181.111.86



;; AUTHORITY SECTION:

baidu.com. 63948 IN NS ns2.baidu.com.

baidu.com. 63948 IN NS dns.baidu.com.

baidu.com. 63948 IN NS ns3.baidu.com.

baidu.com. 63948 IN NS ns7.baidu.com.

baidu.com. 63948 IN NS ns4.baidu.com.



;; ADDITIONAL SECTION:

dns.baidu.com. 13284 IN A 202.108.22.220

ns2.baidu.com. 61082 IN A 61.135.165.235

ns3.baidu.com. 61088 IN A 220.181.37.10

ns4.baidu.com. 13284 IN A 220.181.38.10

ns7.baidu.com. 13284 IN A 119.75.219.82



;; Query time: 87 msec

;; SERVER: 127.0.0.1#53(127.0.0.1)

;; WHEN: Mon Nov 03 13:59:33 CST 2014

;; MSG SIZE  rcvd: 256
```


## 输出结果各项意义

```
; <<>> DiG 9.9.5-3-Ubuntu <<>> baidu.com

dig程序的版本号，和要查询的域名

------

;; global options: +cmd

输出到控制台

------

;; Got answer:

下面是返回信息的内容，分为5部分，头部HEADER、虚选项部OPT PSEUDOSECTION、查询部分QUESTION SECTION、回复部分ANSWER SECTION、权威机构部分AUTHORITY SECTION、附加部分ADDITIONAL SECTION

------

;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 7794

;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 5, ADDITIONAL: 6

头部：

opcode 操作码，QUERY，代表是查询操作

status 状态，NOERROR，代表没有错误

id 编号，7794，16bit数字，在dns协议中，通过编号匹配返回和查询。

flags 标志，如果出现就表示有标志，如果不出现就未设置标志：

  qr query，查询标志，代表是查询操作

  rd recursion desired， 代表希望进行递归(recursive)查询操作

  ra recursive available 在返回中设置，代表查询的服务器支持递归(recursive)查询操作。

  aa Authoritative Answer 权威回复，如果查询结果由管理域名的域名服务器而不是缓存服务器提供的，则称为权威回复。

QUERY 查询数，1代表1个查询，对应下面的QUESTION SECTION中的记录数

ANSWER 结果数，3代表有3项结果，对应下面ANSWER SECTION中的记录数

AUTHORITY 权威域名服务器记录数，5代表该域名有5个权威域名服务器，可供域名解析用。对应下面AUTHORITY SECTION

ADDITIONAL 格外记录数，6代表有6项格外记录。对应下面 ADDITIONAL SECTION。

------

;; OPT PSEUDOSECTION:

; EDNS: version: 0, flags:; udp: 4096

待补充

------

;; QUESTION SECTION:

;baidu.com. IN A

查询部分，从左到右各部分意义：

1、要查询的域名，这里是baidu.com.，'.'代表根域名，com顶级域名，baidu二级域名

2、class，要查询信息的类别，IN代表类别为IP协议，即Internet。还有其它类别，比如chaos等，由于现在都是互联网，所以其它基本不用。

3、type，要查询的记录类型，A记录(Address)，代表要查询ipv4地址。AAAA记录，代表要查询ipv6地址。

------

;; ANSWER SECTION:

baidu.com. 76 IN A 123.125.114.144

baidu.com. 76 IN A 220.181.111.85

baidu.com. 76 IN A 220.181.111.86

回应部分，回应都是A记录，A记录从左到右各部分意义：

1、对应的域名，这里是baidu.com.，'.'代表根域名，com顶级域名，baidu二级域名

2、TTL，time ro live，缓存时间，单位秒。76，代表缓存域名服务器，可以在缓存中保存76秒该记录。

3、class，要查询信息的类别，IN代表类别为IP协议，即Internet。还有其它类别，比如chaos等，由于现在都是互联网，所以其它基本不用。

4、type，要查询的记录类型，A记录，代表要查询ipv4地址。AAAA记录，代表要查询ipv6地址。

5、域名对应的ip地址。

------

;; AUTHORITY SECTION:

baidu.com. 63948 IN NS ns2.baidu.com.

baidu.com. 63948 IN NS dns.baidu.com.

baidu.com. 63948 IN NS ns3.baidu.com.

baidu.com. 63948 IN NS ns7.baidu.com.

baidu.com. 63948 IN NS ns4.baidu.com.

权威域名部分，回应都是NS记录(Name Server)，NS记录从左到右各部分意义：

1、对应的域名，这里是baidu.com.，'.'代表根域名，com顶级域名，baidu二级域名

2、TTL，time ro live，缓存时间，单位秒。63948，代表缓存域名服务器，可以在缓存中保存63948秒该记录。

3、class，要查询信息的类别，IN代表类别为IP协议，即Internet。还有其它类别，比如chaos等，由于现在都是互联网，所以其它基本不用。

4、type，要查询的记录类型，NS，Name Server，NS记录，代表该记录描述了域名对应的权威域名解析服务器

5、域名对应域名对应的权威域名解析服务器。由于ns2.baidu.com.是baidu.com.的子域名，而解析子域名，又需要主域名的信息，为了打破这个死循环，需要在下面的额外记录中提供该服务器的ip地址。

------

;; ADDITIONAL SECTION:

dns.baidu.com. 13284 IN A 202.108.22.220

ns2.baidu.com. 61082 IN A 61.135.165.235

ns3.baidu.com. 61088 IN A 220.181.37.10

ns4.baidu.com. 13284 IN A 220.181.38.10

ns7.baidu.com. 13284 IN A 119.75.219.82

额外记录部分，这里都是A记录，A记录从左到右各部分意义：

1、对应的域名，这里是dns.baidu.com.，'.'代表根域名，com顶级域名，baidu二级域名，dns是三级域名。

2、TTL，time ro live，缓存时间，单位秒。13284，代表缓存域名服务器可以在缓存中保存13284秒该记录。

3、class，要查询信息的类别，IN代表类别为IP协议，即Internet。还有其它类别，比如chaos等，由于现在都是互联网，所以其它基本不用。

4、type，要查询的记录类型，A记录，代表要查询ipv4地址。AAAA记录，代表要查询ipv6地址。

5、域名对应的ip地址。

------

;; Query time: 87 msec

查询耗时

------

;; SERVER: 127.0.0.1#53(127.0.0.1)

查询使用的服务器地址和端口

------

;; WHEN: Mon Nov 03 13:59:33 CST 2014

查询的时间

------

;; MSG SIZE  rcvd: 256

回应的大小。收到(rcve, recieved)256字节。

------

non-recursive query，非递归查询，就是这种查询请求：DNS服务器可以提供完整的域名信息（当它是该域名的权威主解析服务器时），或者在不查询其它dns服务器时提供部分域名信息。

------

recursive query，递归查询，就是就是这种查询请求：DNS服务器通过在必要时查询其它域名服务器以提供完整域名信息。
```

## A记录与CNAME
A记录就是把一个域名解析到一个IP地址（Address，特制数字IP地址）而CNAME记录就是把域名解析到另外一个域名

如果有一个服务器（如IP:88.88.88.88），上面挂了50个网站，对应50个域名，全部是做的A记录，如果某天换服务器了，IP变成了99.99.99.99，这时候是不是要去修改50个域名的A记录？
如果最开始我们把50个域名CNAME到某个域名上，如a.com，再把a.com做A记录到88.88.88.88，当IP更换之后，只需要更换a.com的A记录就行了，方便配置。

