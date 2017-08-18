---
title: "Scrapy Settings"
date: 2017-08-18T19:44:00+08:00
subtitle: ""
tags: ["scrapy"]
---

scrapy框架的通用配置与项目相关配置

<!--more-->

先上总体配置图

![](/images/scrapy setting.png)
![](/images/scrapy setting2.png)


# 一、 Analysis

通过日志、统计和telnet等工具分析调试、提高性能等

### Logging

Scrapy 用不同的日志级别，按由低到高级别：`DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`, `SILENT（安静模式）`。 日志记录用`LOG_LEVER`限定， 经常使用`INFO`级别， 因为`DEBUG`日志信息太冗余。

一个非常有用的Scrapy的扩展是`Log Stats`，`Log Stats`可以统计每60s的`item`的数目以及爬取的页面数。 统计频率可以通过`LOGSTATS_INTERVAL`设置， 在开发期间， 可以设置成5s。

日志可以写入文件， 默认是写到控制台， 可以通过`LOG_FILE`配置日志的文件，`LOG_ENABLED`设置是否开启日志， `LOG_STDOUT`设置true， 可以把打印到控制台的信息打印到日志

### Stats

默认情况下，`STATS_DUMP`被启用， 当爬取结束后， 从`Stats Collector`打印统计数据到日志。您可以通过将`DOWNLOADER_STATS`设置为`False`来控制是否为下载器记录统计信息。您还可以通过`DEPTH_STATS`设置来控制是否收集站点深度的统计信息。 有关深度的更多详细信息，请将`DEPTH_STATS_VERBOSE`设置为`True`。 `STATSMAILER_RCPTS`是一个电子邮件的列表（例如，设置为['my@mail.com']），当爬取结束发送邮件。 您不会经常调整这些设置，但它们偶尔可以帮助调试。

### Telnet

Scrapy包括一个内置的telnet控制台，为您提供了运行Scrapy流程的Python shell。 `TELNETCONSOLE_ENABLED`默认启用，而`TELNETCONSOLE_PORT`确定用于连接到控制台的端口。 你可能需要改变它们，以防万一发生。

### 例子 使用telnet

```
> scrapy crawl fast
    ...
    [scrapy] DEBUG: Telnet console listening on 127.0.0.1:6023:6023
    
> telnet localhost 6023
>>>

这样我们可以审查多个组件， 例如engine组件使用engine变量。 为了快速预览状态等， 可以使用est()命令

>>> est()
Execution engine status
time()-engine.start_time
engine.has_capacity()
len(engine.downloader.active)
...
len(engine.slot.inprogress)
...
len(engine.scraper.slot.active)
: 5.73892092705
: False
: 8
: 10 : 2

>>> engine.pause()
>>> engine.unpause()
>>> engine.stop()

```

# 二、 Performance

性能设置可让您根据特定工作负载调整Scrapy。`CONCURRENT_ REQUESTS`设置要同时执行的最大请求数。这大多数保护您的服务器的出站容量，以防您抓取许多不同的网站（domain / IP）。除非有这种情况，否则通常会更新`CONCURRENT_REQUESTS_PER_DOMAIN`和`CONCURRENT_REQUESTS_PER_IP`，这两者分别通过限制每个唯一域名或IP地址的同时请求数来保护远程服务器。如果`CONCURRENT_ REQUESTS_PER_IP`不为零，则`CONCURRENT_REQUESTS_PER_DOMAIN`将被忽略。如果`CONCURRENT_REQUESTS = 16`，平均请求需要四分之一秒，则您的限制为`16 / 0.25 =每秒64个`请求。`CONCURRENT_ITEM`设置来自并行处理每个响应的最大项目数。您可能会发现这种设置方式比看起来不太有用，因为通常每页/请求都有一个Item。默认值为100也是相当随意的。如果将其减少到例如10或1，您可能会看到性能提升取决于项目/请求的数量以及管道的复杂程度。您还将注意到，由于此值是根据请求，如果`CONCURRENT_REQUESTS = 16`的限制，则`CONCURRENT_ITEMS = 100`可能意味着同时尝试写入数据库的最多1600个项目，等等。

对于下载，`DOWNLOAD_TIMEOUT`确定下载程序在取消请求之前等待的时间。 默认情况下，这是180秒，这似乎是过多的（16个并发请求，这意味着对于一个站点的页面/分钟）。 默认情况下，Scrapy将下载之间的延迟设置为零，以最大限度地提高抓取速度。 您可以使用`DOWNLOAD_DELAY`设置修改此内容以应用更保守的下载速度。有一些网站可以将请求频率作为"机器人"行为的指标。 通过设置`DOWNLOAD_DELAY`，您还可以在下载延迟时启用±50％随机数。 您可以通过将`RANDOMIZE_DOWNLOAD_DELAY`设置为`False`来禁用此功能

最后，为了更快的DNS查找，默认情况下，通过`DNSCACHE_ENABLED`设置启用内存中的DNS缓存。

# 三、 Closing

当满足条件时，Scrapy的CloseSpider扩展名会自动停止蜘蛛爬行。 您可以在一段时间后确定蜘蛛关闭，在收到了一些响应之后，或者在发生了一些错误之后， 分别为`CLOSESPIDER_TIMEOUT（以秒为单位）` `CLOSESPIDER_ITEMCOUNT`, `CLOSESPIDER_PAGECOUNT`和`CLOSESPIDER_ERRORCOUNT`设置。 在运行蜘蛛的过程中，通常会在命令行中设置它们：

```
> scrapy crawl fast -s CLOSESPIDER_ITEMCOUNT=10
> scrapy crawl fast -s CLOSESPIDER_PAGECOUNT=10
> scrapy crawl fast -s CLOSESPIDER_TIMEOUT=10
```

# 四、 Http Cache

Scrapy的`HttpCacheMiddleware`组件（默认情况下禁用）为HTTP请求和响应提供了一个低级缓存。 如果启用，缓存存储每个请求及其相应的响应。

# 五、 Crawling style

Scrapy可以让您调整如何选择哪些页面来抓取。 您可以在`DEPTH_LIMIT`设置中设置最大深度，0表示无限制。 可以设置`DEPTH_PRIORITY`，根据深度来分配请求。 最值得注意的是，您可以通过将此值设置为正数并将调度程序的队列从`LIFO`切换到`FIFO`来执行宽度优先爬网：

```
DEPTH_PRIORITY = 1
SCHEDULER_DISK_QUEUE = 'scrapy.squeue.PickleFifoDiskQueue'
SCHEDULER_MEMORY_QUEUE = 'scrapy.squeue.FifoMemoryQueue'
```

`CookieMiddleware`透明地处理所有与`Cookie`相关的操作，其中包括会话跟踪，允许您登录，等等
上。 如果您想要进行更多的"隐身"抓取，可以将`COOKIES_ENABLED`设置为`False`来禁用此功能。 禁用`Cookie`也会轻微减少您使用的带宽。 类似地，`REFERER_ENABLED`设置默认为`True`，启用`RefererMiddleware`，它会填充`Referer`头。 您可以使用`DEFAULT_REQUEST_HEADERS`定制自定义请求头。这对有些网站的反爬虫很有用。


# 六、 Feeds

# 七、 Media Download

Scrapy可以使用`Image Pipeline`下载媒体内容，它也可以将图像转换为不同的格式，生成缩略图，并根据大小放映图像。

`IMAGES_STORE`设置存储图像的目录（使用相对路径将在项目的根文件夹中创建一个目录）。 每个项目的图像的URL应该在其image_urls字段中（这可以被IMAGES_URLS_FIELD设置覆盖），下载的图像的文件名将被设置为一个新的图像字段（这可以被IMAGES_RESULT_FIELD设置覆盖）。 您可以通过设置`IMAGES_MIN_WIDTH`和`IMAGES_MIN_HEIGHT`来缩略图像。 `IMAGES_EXPIRES`决定在缓存过期之前保存图像的天数。 对于缩略图生成，`IMAGES_THUMBS`设置允许您定义一个或多个缩略图与其尺寸一起生成。 例如，您可以让Scrapy为每个下载的图像生成一个图标大小的缩略图和一个中等缩略图。

您可以使用`FileS Pipeline`下载其他媒体文件。 与图像类似，`FILES_STORE`确定文件下载的位置，`FILES_EXPIRES`确定文件保留的天数。 `FILES_URLS_FIELD`和`FILES_RESULT_FIELD`设置与`IMAGES_ *`对应的功能类似。 文件管理和图像管道都可以同时处于活动状态。

为了使用图片功能， 我们需要安装`image`包， `pip install image`。 在`settings.py`配置`Image Pipeline`

```
ITEM_PIPELINES = {
...
  'scrapy.pipelines.images.ImagesPipeline': 1,
}
IMAGES_STORE = 'images'
IMAGES_THUMBS = { 'small': (30, 30) }
```

# 八、 proxying

Scrapy的`HttpProxyMiddleware`组件允许您根据Unix约定使用`http_proxy`，`https_proxy`和`no_proxy`环境变量定义的代理设置。 此组件默认启用

# 九、 Further settings

与项目相关， 见图二

