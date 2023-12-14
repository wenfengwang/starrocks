---
displayed_sidebar: "Chinese"
---

# Memory Management
内存管理

This section briefly introduces memory classification and StarRocks’ methods of managing memory.
本节简要介绍了内存分类和StarRocks管理内存的方法。

## Memory Classification
内存分类

Explanation:
说明：

|   Metric  | Name | Description |
| --- | --- | --- |
|  process   |  Total memory used of BE  | |
|  query\_pool   |   Memory used by data querying  | 由执行层使用的内存和存储层使用的内存组成。|
|  load   |  Memory used by data loading    | 通常是MemTable|
|  table_meta   |   Metadata memory | S Schema, Tablet元数据, RowSet元数据, Column元数据, ColumnReader, IndexReader |
|  compaction   |   Multi-version memory compaction  |  数据导入完成后发生的压实|
|  snapshot  |   Snapshot memory  | 通常用于clone，占用内存较小 |
|  column_pool   |    Column pool memory   | 请求释放加速列的列缓存 |
|  page_cache   |   BE's own PageCache   | 默认关闭，用户可以通过修改BE文件打开它。|

## 内存相关配置

* **BE配置**

| Name | Default| Description|  
| --- | --- | --- |
| vector_chunk_size | 4096 | chunk行数 |
| mem_limit | 80% | BE可使用的总内存比例。如果BE是独立部署的，则无需配置。如果与消耗更多内存的其他服务一起部署，则应分开配置。 |
| disable_storage_page_cache | false | 控制是否禁用PageCache的布尔值。启用PageCache后，StarRocks缓存最近扫描的数据。当频繁重复相似的查询时，PageCache可以显著提高查询性能。`true`表示禁用PageCache。与`storage_page_cache_limit`一起使用，可以加速具有足够内存资源和大量数据扫描的场景中的查询性能。此项的默认值已从`true`更改为`false`，自StarRocks v2.4起。 |
| write_buffer_size | 104857600 |  单个MemTable的容量限制，超过该限制将执行磁盘擦除。 |
| load_process_max_memory_limit_bytes | 107374182400 | BE节点上所有加载过程占用的内存资源的上限。其值是`mem_limit * load_process_max_memory_limit_percent / 100`和`load_process_max_memory_limit_bytes`中较小的一个。如果超过此阈值，将触发flush和backpressure。  |
| load_process_max_memory_limit_percent | 30 | BE节点上所有加载过程可以占用的最大内存资源百分比。其值是`mem_limit * load_process_max_memory_limit_percent / 100`和`load_process_max_memory_limit_bytes`中较小的一个。如果超过此阈值，将触发flush和backpressure。 |
| default_load_mem_limit | 2147483648 | 如果单个导入实例的接收方的内存限制已达到，将触发磁盘擦除。这需要使用会话变量`load_mem_limit`修改才能生效。 |
| max_compaction_concurrency | -1 | 压实的最大并发数（基础压实和累积压实均适用）。值-1表示并发数没有限制。 |
| cumulative_compaction_check_interval_seconds | 1 | 压实检查间隔|

* **会话变量**

| Name| Default| Description|
| --- | --- | --- |
| query_mem_limit| 0| 每个后端节点上查询的内存限制 |
| load_mem_limit | 0| 单个导入任务的内存限制。如果值为0，将采用`exec_mem_limit`|

## 查看内存使用情况

* **内存追踪器**

~~~ bash
// 查看整体内存统计
<http://be_ip:be_http_port/mem_tracker>

// 查看细粒度的内存统计
<http://be_ip:be_http_port/mem_tracker?type=query_pool&upper_level=3>
~~~

* **tcmalloc**

~~~ bash
<http://be_ip:be_http_port/memz>
~~~

~~~plain text
------------------------------------------------
MALLOC:      777276768 (  741.3 MiB) 应用程序使用的字节
MALLOC: +   8851890176 ( 8441.8 MiB) 页堆空闲列表中的字节
MALLOC: +    143722232 (  137.1 MiB) 中央缓存空闲列表中的字节
MALLOC: +     21869824 (   20.9 MiB) 传输缓存空闲列表中的字节
MALLOC: +    832509608 (  793.9 MiB) 线程缓存自由列表中的字节
MALLOC: +     58195968 (   55.5 MiB) malloc元数据中的字节
MALLOC:   ------------
MALLOC: =  10685464576 (10190.5 MiB) 实际内存使用量（物理内存+交换空间）
MALLOC: +  25231564800 (24062.7 MiB) 释放给操作系统的字节（也称为取消映射）
MALLOC:   ------------
MALLOC: =  35917029376 (34253.1 MiB) 使用的虚拟地址空间
MALLOC:
MALLOC:         112388              使用的跨度
MALLOC:            335              使用的线程堆
MALLOC:           8192              Tcmalloc页大小
------------------------------------------------
调用ReleaseFreeMemory()以释放空闲列表内存给操作系统（通过madvise()）。释放给操作系统的字节占用虚拟地址空间但不占用物理内存。|
~~~

通过此方法查询的内存是准确的。但是在StarRocks中，一些内存是保留但未使用的。TcMalloc计算保留但未使用的内存，而不是已使用的内存。

这里的`应用程序使用的字节`是指当前正在使用的内存。

* **指标**

~~~bash
curl -XGET http://be_ip:be_http_port/metrics | grep 'mem'
curl -XGET http://be_ip:be_http_port/metrics | grep 'column_pool'
~~~

指标的值每隔10秒更新一次。较旧版本可能可以监视一些内存统计信息。

