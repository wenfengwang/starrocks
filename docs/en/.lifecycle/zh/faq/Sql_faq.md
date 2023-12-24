---
displayed_sidebar: English
---

# SQL

本主题提供了有关 SQL 的一些常见问题的答案。

## 当我构建一个物化视图时出现 "无法分配内存" 错误

要解决此问题，请增加 **be.conf** 文件中 `memory_limitation_per_thread_for_schema_change` 参数的值。该参数指的是可以为单个任务分配的最大存储空间，最大存储空间的默认值为 2 GB。

## StarRocks 是否支持缓存查询结果？

StarRocks 不直接缓存最终查询结果。从 v2.5 开始，StarRocks 使用查询缓存功能将第一阶段聚合的中间结果保存在缓存中。与之前的查询语义相同的新查询可以重用缓存的计算结果以加速计算。查询缓存使用 BE 内存。有关更多信息，请参见 [查询缓存](../using_starrocks/query_cache.md)。

## 当计算中包含 `Null` 时，除了 ISNULL() 函数外，函数的计算结果都是 false

在标准 SQL 中，每个包含值为 `NULL` 的操作数的计算都会返回 `NULL`。

## 在对 BIGINT 数据类型的值进行等效查询时，如果将其用引号括起来，为什么查询结果不正确？

### 问题描述

请参阅以下示例：

```plaintext
select cust_id,idno 

from llyt_dev.dwd_mbr_custinfo_dd 

where Pt= ‘2021-06-30’ 

and cust_id = ‘20210129005809043707’ 

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

where Pt= ‘2021-06-30’ 

and cust_id = 20210129005809043707 

limit 10 offset 0;
+---------------------+-----------------------------------------+

|   cust_id           |      idno                               |

+---------------------+-----------------------------------------+

|  20210189979989976  | xuywehuhfuhruehfurhghcfCNMX39dw=        |

+---------------------+-----------------------------------------+
```

### 解决方案

当比较 STRING 数据类型和 INTEGER 数据类型时，这两种类型的字段都会被强制转换为 DOUBLE 数据类型。因此，不能添加引号。否则，无法对 WHERE 子句中定义的条件进行索引。

## StarRocks 是否支持 DECODE 函数？

StarRocks 不支持 Oracle 数据库的 DECODE 函数。StarRocks 兼容 MySQL，因此可以使用 CASE WHEN 语句。

## 在将数据加载到 StarRocks 的主键表后，是否可以立即查询最新数据？

是的。StarRocks 以引用 Google Mesa 的方式合并数据。在 StarRocks 中，BE 触发数据合并，它有两种合并数据的方式。如果数据合并尚未完成，则在查询期间完成。因此，在加载数据后，您可以读取最新数据。

## 存储在 StarRocks 中的 utf8mb4 字符会被截断或出现乱码吗？

不会。

## 运行 `alter table` 命令时出现 "表的状态不正常" 错误

出现此错误是因为先前的更改尚未完成。您可以运行以下代码来检查上一次更改的状态：

```SQL
show tablet from lineitem where State="ALTER"; 
```

更改操作所花费的时间与数据量有关。一般来说，更改可以在几分钟内完成。我们建议您在更改表结构时停止向 StarRocks 加载数据，因为数据加载会降低更改完成的速度。

## 查询 Apache Hive 的外部表时出现 "获取分区详细信息失败：org.apache.doris.common.DdlException: 获取hive分区元数据失败: java.net.UnknownHostException:hadooptest" 错误

出现此错误是因为无法获取 Apache Hive 分区的元数据。要解决此问题，请将 **core-sit.xml** 和 **hdfs-site.xml** 复制到 **fe.conf** 文件和 **be.conf** 文件中。

## 查询数据时出现 "planner use long time 3000 remaining task num 1" 错误

通常出现此错误是因为进行了完整的垃圾回收（full GC），可以通过后端监控和 **fe.gc** 日志进行检查。要解决此问题，请执行以下操作之一：

- 允许 SQL 的客户端同时访问多个前端 (FE) 以分散负载。
- 在 **fe.conf** 文件中将 Java 虚拟机 (JVM) 的堆大小从 8 GB 更改为 16 GB，以增加内存并减少完整 GC 的影响。

## 当 A 列的基数较小时，每次的查询结果 `select B from tbl order by A limit 10` 都会有所不同

SQL 只能保证 A 列是有序的，不能保证每次查询的 B 列顺序相同。MySQL 可以保证 A 列和 B 列的顺序，因为它是一个独立的数据库。

StarRocks 是一个分布式数据库，底层表中存储的数据是分片模式的。A 列的数据分布在多台机器上，因此每次查询多台机器返回的 B 列顺序可能不同，导致每次 B 的顺序不一致。要解决此问题，请将 `select B from tbl order by A limit 10` 更改为 `select B from tbl order by A,B limit 10`。

## SELECT * 和 SELECT 在效率上存在较大差距的原因是什么？

要解决此问题，请检查配置文件并查看 MERGE 的详细信息：

- 检查存储层的聚合是否占用了太多时间。
- 检查是否有太多的指标列。如果有太多的指标列，那么对数百万行的数百列进行聚合会花费很长时间。

```plaintext
MERGE:

    - aggr: 26s270ms

    - sort: 15s551ms
```

## DELETE 支持嵌套函数吗？

不支持嵌套函数，例如 `DELETE from test_new WHERE to_days(now())-to_days(publish_time) >7;` 中的 `to_days(now())`。

## 当数据库中有数百个表时，如何提高其使用效率？

为了提高效率，在连接到 MySQL 的客户端服务器时，请添加 `-A` 参数：`mysql -uroot -h127.0.0.1 -P8867 -A`。MySQL 的客户端服务器不会预读取数据库信息。

## 如何减少 BE 日志和 FE 日志占用的磁盘空间？

调整日志级别和相应的参数。有关更多信息，请参见 [参数配置](../administration/BE_configuration.md)。

## 修改复制编号时出现 "table *** is colocate table， cannot change replicationNum" 错误

创建共置表时，需要设置 `group` 属性。因此，您不能修改单个表的复制编号。您可以按照以下步骤修改组内所有表的复制编号：

1.  将组中所有表的 `group_with` 设置为 `empty`。
2.  为组中所有表设置适当的 `replication_num`。
3.  将 `group_with` 设置回其原始值。

## 将 VARCHAR 设置为最大值会影响存储吗？

VARCHAR 是一种可变长度数据类型，其指定长度可以根据实际数据长度进行更改。在创建表时指定不同的 varchar 长度对相同数据的查询性能影响不大。

## 截断表时出现 "create partititon timeout" 错误

要截断表，您需要创建相应的分区，然后交换它们。如果需要创建更多的分区，则会出现此错误。此外，如果有许多数据加载任务，则在压缩过程中会长时间保持锁定。因此，在创建表时无法获取锁。如果有太多的数据加载任务，请在 **be.conf** 文件中将 `tablet_map_shard_size` 设置为 `512` 以减少锁争用。

## 访问 Apache Hive 的外部表时出现 "Failed to specify server's Kerberos principal name" 错误

请将以下信息添加到 **fe.conf** 文件和 **be.conf** 文件的 **hdfs-site.xml** 中：

```HTML
<property>

<name>dfs.namenode.kerberos.principal.pattern</name>

<value>*</value>

</property>
```

## "2021-10" 是 StarRocks 中的日期格式吗？

不是。

## "2021-10" 可以用作分区字段吗？

不可以，应使用函数将 "2021-10" 更改为 "2021-10-01"，然后将 "2021-10-01" 用作分区字段。

## 在哪里可以查询 StarRocks 数据库或表的大小？

您可以使用 [SHOW DATA](../sql-reference/sql-statements/data-manipulation/SHOW_DATA.md) 命令。

`SHOW DATA;` 显示当前数据库中所有表的数据大小和副本数。

`SHOW DATA FROM <db_name>.<table_name>;` 显示指定数据库中指定表的数据大小、副本数和行数。
