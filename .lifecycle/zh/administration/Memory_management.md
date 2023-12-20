---
displayed_sidebar: English
---

# 内存管理

本节简要介绍内存的分类及StarRocks如何管理内存。

## 内存分类

说明：

|指标|名称|描述|
|---|---|---|
|进程|BE 使用的总内存|
|query\_pool|数据查询使用的内存|由两部分组成：执行层使用的内存和存储层使用的内存。|
|load|数据加载使用的内存|一般为MemTable|
|table_meta|元数据内存|S Schema、Tablet 元数据、RowSet 元数据、Column 元数据、ColumnReader、IndexReader|
|compaction|多版本内存压缩|数据导入完成后进行压缩|
|snapshot|快照内存|一般用于克隆，占用内存很少|
|column_pool|列池内存|请求释放加速列的列缓存|
|page_cache|BE自带的PageCache|默认关闭，用户可以通过修改BE文件开启|

## 内存相关配置

* **BE配置**

|名称|默认|描述|
|---|---|---|
|vector_chunk_size|4096|块行数|
|mem_limit|80%|BE 可以使用的总内存的百分比。如果BE独立部署，则无需配置。如果与其他消耗内存较多的服务一起部署，则需要单独配置。|
|disable_storage_page_cache|false|控制是否禁用PageCache的布尔值。启用 PageCache 后，StarRocks 会缓存最近扫描的数据。当频繁重复类似查询时，PageCache 可以显着提高查询性能。 true 表示禁用 PageCache。此项与storage_page_cache_limit配合使用，可以在内存资源充足、数据扫描较多的场景下加快查询性能。自 StarRocks v2.4 起，此项的默认值已从 true 更改为 false。|
|write_buffer_size|104857600|单个MemTable的容量限制，超过该容量将执行磁盘刷卡操作。|
|load_process_max_memory_limit_bytes|107374182400|BE节点上所有加载进程可以占用的内存资源上限。其值是 mem_limit * load_process_max_memory_limit_percent / 100 和 load_process_max_memory_limit_bytes 之间较小的一个。如果超过此阈值，将触发冲洗和背压。|
|load_process_max_memory_limit_percent|30|BE节点上所有加载进程可以占用的内存资源的最大百分比。其值是 mem_limit * load_process_max_memory_limit_percent / 100 和 load_process_max_memory_limit_bytes 之间较小的一个。如果超过此阈值，将触发冲洗和背压。|
|default_load_mem_limit|2147483648|如果单个导入实例达到接收端的内存限制，将触发磁盘刷卡。这需要使用Session变量load_mem_limit进行修改才能生效。|
|max_compaction_concurrency|-1|压缩的最大并发度（基础压缩和累积压缩）。 -1表示不限制并发数。|
|cumulative_compaction_check_interval_seconds|1|压缩检查的间隔|

* **会话变量**

|名称|默认|描述|
|---|---|---|
|query_mem_limit|0|每个后端节点查询的内存限制|
|load_mem_limit|0|单个导入任务的内存限制。如果值为0，将采用exec_mem_limit |

## 查看内存使用情况

* **mem\_tracker**

```bash
//View the overall memory statistics
<http://be_ip:be_http_port/mem_tracker>

// View fine-grained memory statistics
<http://be_ip:be_http_port/mem_tracker?type=query_pool&upper_level=3>
```

* **tcmalloc**

```bash
<http://be_ip:be_http_port/memz>
```

```plain
------------------------------------------------
MALLOC:      777276768 (  741.3 MiB) Bytes in use by application
MALLOC: +   8851890176 ( 8441.8 MiB) Bytes in page heap freelist
MALLOC: +    143722232 (  137.1 MiB) Bytes in central cache freelist
MALLOC: +     21869824 (   20.9 MiB) Bytes in transfer cache freelist
MALLOC: +    832509608 (  793.9 MiB) Bytes in thread cache freelists
MALLOC: +     58195968 (   55.5 MiB) Bytes in malloc metadata
MALLOC:   ------------
MALLOC: =  10685464576 (10190.5 MiB) Actual memory used (physical + swap)
MALLOC: +  25231564800 (24062.7 MiB) Bytes released to OS (aka unmapped)
MALLOC:   ------------
MALLOC: =  35917029376 (34253.1 MiB) Virtual address space used
MALLOC:
MALLOC:         112388              Spans in use
MALLOC:            335              Thread heaps in use
MALLOC:           8192              Tcmalloc page size
------------------------------------------------
Call ReleaseFreeMemory() to release freelist memory to the OS (via madvise()).
Bytes released to the OS take up virtual address space but no physical memory.
```

此方法查询到的内存数据是准确的。然而，StarRocks中有些内存虽然已预留但尚未使用。TcMalloc统计的是已预留的内存，而非实际使用的内存。

这里的“应用程序使用中的字节”指的是当前实际使用的内存。

* **指标**

```bash
curl -XGET http://be_ip:be_http_port/metrics | grep 'mem'
curl -XGET http://be_ip:be_http_port/metrics | grep 'column_pool'
```

指标值每10秒更新一次。可以用来监控一些旧版本的内存统计数据。
