---
displayed_sidebar: English
---

# 内存管理

本节简要介绍内存分类以及StarRocks如何管理内存。

## 内存分类

说明：

|指标|名称|描述|
|---|---|---|
|process|BE使用的总内存|
|query\_pool|数据查询使用的内存|包括两部分：执行层使用的内存和存储层使用的内存。|
|load|数据加载使用的内存|通常是MemTable|
|table_meta|元数据内存|包括Schema、Tablet元数据、RowSet元数据、列元数据、ColumnReader、IndexReader|
|compaction|多版本内存压缩|数据导入完成后的压缩操作|
|snapshot|快照内存|通常用于克隆操作，内存使用量很小|
|column_pool|列池内存|请求释放列缓存以加速列访问|
|page_cache|BE自身的PageCache|默认关闭，用户可以通过修改BE配置文件来开启|

## 内存相关配置

* **BE配置**

|名称|默认值|描述|
|---|---|---|
|vector_chunk_size|4096|Chunk的行数|
|mem_limit|80%|BE可以使用的总内存百分比。如果BE独立部署，无需配置此项。如果与其他内存消耗较大的服务共同部署，则应单独配置。|
|disable_storage_page_cache|false|控制是否禁用PageCache的布尔值。启用PageCache后，StarRocks会缓存最近扫描的数据。当类似的查询频繁重复时，PageCache可以显著提升查询性能。`true`表示禁用PageCache。与`storage_page_cache_limit`一起使用时，可以在内存资源充足且数据扫描频繁的场景下提升查询性能。自StarRocks v2.4起，默认值已从`true`更改为`false`。|
|write_buffer_size|104857600|单个MemTable的容量上限，超出此容量将进行磁盘写入操作。|
|load_process_max_memory_limit_bytes|107374182400|BE节点上所有加载进程可占用的最大内存资源。其值为`mem_limit * load_process_max_memory_limit_percent / 100`与`load_process_max_memory_limit_bytes`中的较小者。超过此阈值时，将触发刷新和背压机制。|
|load_process_max_memory_limit_percent|30|BE节点上所有加载进程可占用的最大内存资源百分比。其值为`mem_limit * load_process_max_memory_limit_percent / 100`与`load_process_max_memory_limit_bytes`中的较小者。超过此阈值时，将触发刷新和背压机制。|
|default_load_mem_limit|2147483648|单个导入实例达到接收端内存限制时，将触发磁盘写入。需与会话变量`load_mem_limit`一同修改才能生效。|
|max_compaction_concurrency|-1|压缩操作的最大并发数（包括基础压缩和累积压缩）。值-1表示不限制并发数。|
|cumulative_compaction_check_interval_seconds|1|压缩检查间隔时间|

* **会话变量**

|名称|默认值|描述|
|---|---|---|
|query_mem_limit|0|每个后端节点上查询的内存限制|
|load_mem_limit|0|单个导入任务的内存限制。如果值为0，则使用`exec_mem_limit`的值|

## 查看内存使用情况

* **mem\_tracker**

```bash
// 查看总体内存统计信息
<http://be_ip:be_http_port/mem_tracker>

// 查看细粒度内存统计信息
<http://be_ip:be_http_port/mem_tracker?type=query_pool&upper_level=3>
```

* **tcmalloc**

```bash
<http://be_ip:be_http_port/memz>
```

```plain
------------------------------------------------
MALLOC:      777276768 (  741.3 MiB) 应用程序使用的内存
MALLOC: +   8851890176 ( 8441.8 MiB) 页面堆空闲列表中的字节
MALLOC: +    143722232 (  137.1 MiB) 中心缓存空闲列表中的字节
MALLOC: +     21869824 (   20.9 MiB) 传输缓存空闲列表中的字节
MALLOC: +    832509608 (  793.9 MiB) 线程缓存空闲列表中的字节
MALLOC: +     58195968 (   55.5 MiB) malloc元数据中的字节
MALLOC:   ------------
MALLOC: =  10685464576 (10190.5 MiB) 实际使用的内存（物理内存 + 交换空间）
MALLOC: +  25231564800 (24062.7 MiB) 释放给操作系统的字节（即未映射）
MALLOC:   ------------
MALLOC: =  35917029376 (34253.1 MiB) 使用的虚拟地址空间
MALLOC:
MALLOC:         112388              使用中的跨度
MALLOC:            335              使用中的线程堆
MALLOC:           8192              Tcmalloc页面大小
------------------------------------------------
调用ReleaseFreeMemory()将空闲列表内存释放给操作系统（通过madvise()）。
释放给操作系统的字节占用虚拟地址空间，但不占用物理内存。
```

该方法查询的内存是准确的。然而，StarRocks中有些内存虽然已预留但未使用。TcMalloc计算的是预留的内存，而非实际使用的内存。

这里的“应用程序使用的内存”指的是当前实际使用的内存。

* **指标**

```bash
curl -XGET http://be_ip:be_http_port/metrics | grep 'mem'
curl -XGET http://be_ip:be_http_port/metrics | grep 'column_pool'
```

指标值每10秒更新一次。可以用来监控一些旧版本的内存统计信息。