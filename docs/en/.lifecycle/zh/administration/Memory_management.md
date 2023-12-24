---
displayed_sidebar: English
---

# 内存管理

本节简要介绍内存分类以及 StarRocks 管理内存的方法。

## 内存分类

解释：

|   指标  | 名称 | 描述 |
| --- | --- | --- |
|  process   |  BE 总内存使用量  | |
|  query\_pool   |   数据查询使用的内存  | 由执行层使用的内存和存储层使用的内存两部分组成。|
|  load   |  数据加载使用的内存    | 通常是 MemTable|
|  table_meta   |   元数据内存 | S Schema、Tablet 元数据、RowSet 元数据、Column 元数据、ColumnReader、IndexReader |
|  compaction   |   多版本内存压缩  | 数据导入完成后进行的压实 |
|  snapshot  |   快照内存  | 一般用于克隆，内存占用较小 |
|  column_pool   |    列池内存   | 请求释放加速列的列缓存 |
|  page_cache   |   BE 自身的 PageCache   | 默认为关闭，用户可以通过修改 BE 文件来打开它 |

## 内存相关配置

* **BE 配置**

| 名称 | 默认值| 描述|  
| --- | --- | --- |
| vector_chunk_size | 4096 | 块行数 |
| mem_limit | 80% | BE 可使用的总内存百分比。如果 BE 作为独立部署，则无需配置。如果与其他消耗更多内存的服务一起部署，应单独配置。 |
| disable_storage_page_cache | false | 用于控制是否禁用 PageCache 的布尔值。启用 PageCache 后，StarRocks 会缓存最近扫描的数据。当频繁重复类似的查询时，PageCache 可显著提高查询性能。`true` 表示禁用 PageCache。与 `storage_page_cache_limit` 一起使用，可在内存资源充足、数据扫描量大的场景下提高查询性能。自 StarRocks v2.4 起，此项的默认值已从 `true` 更改为 `false`。|
| write_buffer_size | 104857600 |  单个 MemTable 的容量限制，超过该限制将执行磁盘写入。 |
| load_process_max_memory_limit_bytes | 107374182400 | BE 节点上所有加载进程可占用的内存资源上限。其值为 `mem_limit * load_process_max_memory_limit_percent / 100` 和 `load_process_max_memory_limit_bytes` 之间的较小值。如果超过此阈值，将触发冲刷和背压。  |
| load_process_max_memory_limit_percent | 30 | BE 节点上所有加载进程可占用的内存资源的最大百分比。其值为 `mem_limit * load_process_max_memory_limit_percent / 100` 和 `load_process_max_memory_limit_bytes` 之间的较小值。如果超过此阈值，将触发冲刷和背压。 |
| default_load_mem_limit | 2147483648 | 如果单个导入实例达到接收端的内存限制，则将触发磁盘写入。需使用会话变量 `load_mem_limit` 进行修改才能生效。 |
| max_compaction_concurrency | -1 | 压实的最大并发数（基本压实和累积压实）。值 -1 表示并发数不受限制。 |
| cumulative_compaction_check_interval_seconds | 1 | 压实检查间隔|

* **会话变量**

| 名称| 默认值| 描述|
| --- | --- | --- |
| query_mem_limit| 0| 每个后端节点上查询的内存限制 |
| load_mem_limit | 0| 单个导入任务的内存限制。若值为 0，则取 `exec_mem_limit`|

## 查看内存使用情况

* **内存追踪器**

~~~ bash
// 查看整体内存统计
<http://be_ip:be_http_port/mem_tracker>

// 查看细粒度内存统计
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
MALLOC: +    832509608 (  793.9 MiB) 线程缓存空闲列表中的字节
MALLOC: +     58195968 (   55.5 MiB) malloc 元数据中的字节
MALLOC:   ------------
MALLOC: =  10685464576 (10190.5 MiB) 实际使用的内存（物理 + 交换）
MALLOC: +  25231564800 (24062.7 MiB) 释放给操作系统的字节（也称为未映射）
MALLOC:   ------------
MALLOC: =  35917029376 (34253.1 MiB) 已使用的虚拟地址空间
MALLOC:
MALLOC:         112388              使用中的跨度
MALLOC:            335              使用中的线程堆
MALLOC:           8192              Tcmalloc 页大小
------------------------------------------------
调用 ReleaseFreeMemory() 以将空闲列表内存释放给操作系统（通过 madvise()）。
释放给操作系统的字节占用虚拟地址空间，但不占用物理内存。
~~~

该方法查询的内存是准确的。然而，在 StarRocks 中，一些内存是预留的，但未被使用。TcMalloc 计算的是已预留的内存，而不是已使用的内存。

这里的 `应用程序使用的字节` 指的是当前正在使用的内存。

* **指标**

~~~bash
curl -XGET http://be_ip:be_http_port/metrics | grep 'mem'
curl -XGET http://be_ip:be_http_port/metrics | grep 'column_pool'
~~~

指标的值每 10 秒更新一次。在旧版本中，可以监视一些内存统计信息。
