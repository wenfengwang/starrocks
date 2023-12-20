---
displayed_sidebar: English
---

# SQL

本主题提供有关 SQL 的一些常见问题的解答。

## 此错误“无法分配内存。”当我构建物化视图时

要解决此问题，请增加 **be.conf** 文件中 `memory_limitation_per_thread_for_schema_change` 参数的值。该参数指的是单个任务进行模式更改时可以分配的最大存储空间。默认的最大存储值为 2 GB。

## StarRocks 支持缓存查询结果吗？

StarRocks 不直接缓存最终查询结果。从 v2.5 版本开始，StarRocks 使用查询缓存功能来保存第一阶段聚合的中间结果到缓存中。新的查询如果在语义上等同于之前的查询，可以重用缓存中的计算结果来加速计算。查询缓存使用 BE 的内存。更多信息，请参见[查询缓存](../using_starrocks/query_cache.md)。

## 当计算中包含 `Null` 时，除了 ISNULL() 函数外，函数的计算结果都是错误的

在标准 SQL 中，任何包含 `NULL` 值操作数的计算都将返回 `NULL`。

## 为什么在 BIGINT 数据类型的值外加上引号后，等值查询的结果不正确？

### 问题描述

参见以下示例：

```plaintext
select cust_id,idno 

from llyt_dev.dwd_mbr_custinfo_dd 

where Pt= '2021-06-30' 

and cust_id = '20210129005809043707' 

limit 10 offset 0;
+---------------------+-----------------------------------------+

|   cust_id           |      idno                               |

+---------------------+-----------------------------------------+

|  20210129005809436  | yjdgjwsnfmdhjw294F93kmHCNMX39dw=        |

|  20210129005809436  | sdhnswjwijeifme3kmHCNMX39gfgrdw=        |

|  20210129005809436  | Tjoedk3js82nswndrf43X39hbggggbw=        |

|  20210129005809436  | denuwjaxh73e39592jwshbnjdi22ogw=        |

|  20210129005809436  | ckxwmsd2mei3nrunjrihj93dm3ijin2=        |

|  20210129005809436  | djm2emdi3mfi3mfu4jro2ji2ndimi3n=        |

+---------------------+-----------------------------------------+
select cust_id,idno 

from llyt_dev.dwd_mbr_custinfo_dd 

where Pt= '2021-06-30' 

and cust_id = 20210129005809043707 

limit 10 offset 0;
+---------------------+-----------------------------------------+

|   cust_id           |      idno                               |

+---------------------+-----------------------------------------+

|  20210189979989976  | xuywehuhfuhruehfurhghcfCNMX39dw=        |

+---------------------+-----------------------------------------+
```

### 解决方案

当比较 STRING 数据类型和 INTEGER 数据类型时，这两种类型的字段会被转换为 DOUBLE 数据类型。因此，不应添加引号。否则，WHERE 子句中定义的条件将无法被索引。

## StarRocks 支持 DECODE 函数吗？

StarRocks 不支持 Oracle 数据库的 DECODE 函数。StarRocks 兼容 MySQL，因此您可以使用 CASE WHEN 语句。

## 在 StarRocks 主键表中加载数据后，可以立即查询到最新数据吗？

是的。StarRocks 采用了类似 Google Mesa 的数据合并方式。在 StarRocks 中，BE 触发数据合并，并且有两种压缩方式来合并数据。如果数据合并未完成，它会在查询期间完成。因此，数据加载后您可以读取到最新数据。

## StarRocks 中存储的 utf8mb4 字符会被截断或显示乱码吗？

不会。

## 当我运行 `alter table` 命令时出现“表的状态不正常”错误

这个错误是因为之前的修改操作尚未完成。您可以运行以下代码来检查之前修改的状态：

```SQL
show tablet from lineitem where State="ALTER"; 
```

修改操作所需的时间与数据量有关。通常，修改可以在几分钟内完成。我们建议您在对表进行修改时停止向 StarRocks 加载数据，因为数据加载会降低修改完成的速度。

## 当我查询 Apache Hive 的外部表时出现“获取分区详细信息失败：org.apache.doris.common.DdlException：获取 hive 分区元数据失败：java.net.UnknownHostException:hadooptest”错误

当无法获取 Apache Hive 分区的元数据时，会出现此错误。为了解决这个问题，请将 **core-site.xml** 和 **hdfs-site.xml** 复制到 **fe.conf** 和 **be.conf** 文件中。

## 当我查询数据时出现“planner use long time 3000 remaining task num 1”错误

这个错误通常是由于全面垃圾收集（full GC）导致的，可以通过后端监控和 **fe.gc** 日志来检查。为了解决这个问题，执行以下操作之一：

- 允许 SQL 客户端同时访问多个前端（FEs）以分散负载。
- 在 **fe.conf** 文件中将 Java 虚拟机（JVM）的堆大小从 8 GB 增加到 16 GB，以增加内存并减少 full GC 的影响。

## 当列 A 的基数较小时，`select B from tbl order by A limit 10` 的查询结果每次都不同

SQL 只能保证列 A 是有序的，并不能保证每次查询时列 B 的顺序相同。MySQL 可以保证列 A 和列 B 的顺序，因为它是一个单机数据库。

StarRocks 是一个分布式数据库，其底层表的数据以分片模式存储。列 A 的数据分布在多台机器上，所以每次查询时多台机器返回的列 B 的顺序可能不同，导致每次 B 的顺序不一致。为了解决这个问题，请将 `select B from tbl order by A limit 10` 改为 `select B from tbl order by A,B limit 10`。

## 为什么 SELECT * 和 SELECT 的列效率之间存在很大差距？

要解决这个问题，请检查 profile 并查看 MERGE 的详细信息：

- 检查存储层的聚合是否占用了太多时间。

- 检查是否有太多的指标列。如果是，请对数百万行的数百列进行聚合。

```plaintext
MERGE:

    - aggr: 26s270ms

    - sort: 15s551ms
```

## DELETE 支持嵌套函数吗？

不支持嵌套函数，例如 `DELETE from test_new WHERE to_days(now())-to_days(publish_time) > 7;` 中的 `to_days(now())`。

## 当数据库中有数百个表时，如何提高数据库的使用效率？

为了提高效率，在连接 MySQL 客户端服务器时添加 `-A` 参数：`mysql -uroot -h127.0.0.1 -P8867 -A`。MySQL 客户端服务器不会预读数据库信息。

## 如何减少 BE 日志和 FE 日志占用的磁盘空间？

调整日志级别和相应的参数。更多信息，请参见[参数配置](../administration/BE_configuration.md)。

## 当我修改复制数时出现“table *** is colocate table, cannot change replicationNum”错误

当您创建共位表时，需要设置 `group` 属性。因此，您不能修改单个表的复制数。您可以按照以下步骤修改组中所有表的复制数：

1. 将组中所有表的 `group_with` 设置为 `empty`。
2. 为组中所有表设置合适的 `replication_num`。
3. 将 `group_with` 恢复为原始值。

## 将 VARCHAR 设置为最大值是否会影响存储？

VARCHAR 是一种变长数据类型，它有一个指定的长度，可以根据实际数据长度变化。在创建表时指定不同的 VARCHAR 长度对于相同数据的查询性能影响很小。

## 当我截断表时出现“创建分区超时”错误

要截断表，您需要创建相应的分区然后交换它们。如果需要创建的分区数量较多，就会出现这个错误。此外，如果有很多数据加载任务，在压缩过程中锁会被长时间持有，因此在创建表时无法获得锁。如果数据加载任务过多，可以在 **be.conf** 文件中将 `tablet_map_shard_size` 设置为 `512`，以减少锁竞争。

## 当我访问 Apache Hive 的外部表时出现“Failed to specify server's Kerberos principal name”错误

在 **fe.conf** 和 **be.conf** 文件中的 **hdfs-site.xml** 中添加以下信息：

```HTML
<property>

<name>dfs.namenode.kerberos.principal.pattern</name>

<value>*</value>

</property>
```

## “2021-10” 是 StarRocks 中的日期格式吗？

不是。

## “2021-10” 可以作为分区字段吗？

不能，使用函数将 “2021-10” 转换为 “2021-10-01”，然后使用 “2021-10-01” 作为分区字段。

## 我在哪里可以查询 StarRocks 数据库或表的大小？

您可以使用 [SHOW DATA](../sql-reference/sql-statements/data-manipulation/SHOW_DATA.md) 命令。

`SHOW DATA;` 显示当前数据库中所有表的数据大小和副本数。

`SHOW DATA FROM <db_name>.<table_name>;` 显示指定数据库中指定表的数据大小、副本数和行数。