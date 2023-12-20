---
displayed_sidebar: English
---

# SQL

本主题提供了一些关于 SQL 常见问题的解答。

## 出现此错误“无法分配内存”时，是在我尝试构建物化视图。

要解决此问题，请在 **be.conf** 文件中增加 `memory_limitation_per_thread_for_schema_change` 参数的值。此参数指定了单个任务在变更模式时可分配的最大存储量，默认最大值为 2 GB。

## StarRocks 支持缓存查询结果吗？

StarRocks 不直接缓存最终查询结果。从 v2.5 版本开始，StarRocks 使用查询缓存功能来保存第一阶段聚合的中间结果。与先前的查询语义等效的新查询可以重用缓存中的计算结果，以加快计算速度。查询缓存使用 BE 的内存。更多信息请参见[查询缓存](../using_starrocks/query_cache.md)部分。

## 当计算中包含 Null 值时，除了 ISNULL() 函数之外，其他函数的计算结果都是 false

在标准 SQL 中，任何包含 NULL 值操作数的计算结果都将返回 NULL。

## 为什么在进行等值查询时，对 BIGINT 数据类型的值加上引号后，查询结果会不正确？

### 问题描述

参见下列示例：

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

当比较 STRING 数据类型和 INTEGER 数据类型时，这两种类型的字段会被转换成 DOUBLE 数据类型。因此，不应该加上引号，否则 WHERE 子句中定义的条件将无法使用索引。

## StarRocks 支持 DECODE 函数吗？

StarRocks 不支持 Oracle 数据库的 DECODE 函数。StarRocks 与 MySQL 兼容，所以您可以使用 CASE WHEN 语句。

## 在数据加载到 StarRocks 的主键表后，可以立即查询到最新数据吗？

可以。StarRocks 采用类似于 Google Mesa 的方式进行数据合并。在 StarRocks 中，BE 触发数据合并，并且有两种压缩方式来合并数据。如果数据合并未完成，将在查询时完成。因此，数据加载后您可以立即读取最新数据。

## 存储在 StarRocks 中的 utf8mb4 字符会被截断或显示乱码吗？

不会。

## 当我运行 alter table 命令时，出现了“表的状态不正常”的错误

这个错误是因为之前的变更还没有完成。您可以运行以下代码来检查之前变更的状态：

```SQL
show tablet from lineitem where State="ALTER"; 
```

变更操作所需的时间与数据量有关。通常，变更可以在几分钟内完成。我们建议在对表进行变更时暂停向 StarRocks 加载数据，因为数据加载会降低变更完成的速度。

## 当我查询 Apache Hive 的外部表时，出现了“获取分区详细信息失败：org.apache.doris.common.DdlException: 获取 hive 分区元数据失败：java.net.UnknownHostException：hadooptest”的错误

当无法获取 Apache Hive 分区的元数据时会出现这个错误。解决这个问题的方法是，将 **core-site.xml** 和 **hdfs-site.xml** 复制到 **fe.conf** 文件和 **be.conf** 文件中。

## 查询数据时出现了“planner use long time 3000 remaining task num 1”的错误

这个错误通常是由于完全垃圾回收（full GC）引起的，可以通过后端监控和 **fe.gc** 日志来进行检查。解决这个问题，可以采取以下操作之一：

- 允许 SQL 客户端同时访问多个前端（FEs），以分散负载。
- 在 **fe.conf** 文件中将 Java 虚拟机（JVM）的堆大小从 8 GB 增加到 16 GB，以增加内存并减少 full GC 的影响。

## 当列 A 的基数较小时，select B from tbl order by A limit 10 的查询结果每次都不同

SQL 只能保证列 A 有序，无法保证每次查询列 B 的顺序相同。MySQL 可以保证列 A 和列 B 的顺序，因为它是一个单机数据库。

StarRocks 是一个分布式数据库，底层表的数据是按分片模式存储的。列 A 的数据分布在多个机器上，因此每次查询时多个机器返回的列 B 顺序可能不同，导致列 B 的顺序每次不一致。解决这个问题，应该将 select B from tbl order by A limit 10 改为 select B from tbl order by A, B limit 10。

## 为什么 SELECT * 和 SELECT 的列效率差距这么大？

要解决这个问题，请检查 Profile 并查看 MERGE 的详细信息：

- 检查存储层的聚合操作是否耗时过长。

- 检查是否有过多的指标列。如果是，请对数百万行的数百列进行聚合操作。

```plaintext
MERGE:

    - aggr: 26s270ms

    - sort: 15s551ms
```

## DELETE 支持嵌套函数吗？

不支持嵌套函数，例如在 DELETE from test_new WHERE to_days(now())-to_days(publish_time) > 7; 中的 to_days(now())。

## 当一个数据库中有数百个表时，如何提高数据库的使用效率？

为了提高效率，在连接 MySQL 客户端服务器时添加 -A 参数：mysql -uroot -h127.0.0.1 -P8867 -A。MySQL 的客户端服务器不会预读数据库信息。

## 如何减少 BE 日志和 FE 日志所占用的磁盘空间？

调整日志级别和相应的参数。更多信息，请参考[参数配置](../administration/BE_configuration.md)。

## 当我修改复制数时，出现了“表 *** 是共置表，无法更改 replicationNum”的错误

创建共置表时，需要设置组属性，因此无法为单个表修改复制数。您可以按照以下步骤来修改组中所有表的复制数：

1. 将组中所有表的 group_with 设置为空。
2. 为组中所有表设置合适的 replication_num。
3. 将 group_with 恢复到原来的值。

## 将 VARCHAR 设置为最大值是否会影响存储？

VARCHAR 是一种可变长度的数据类型，它有一个指定长度，可以根据实际数据长度变化。在创建表时指定不同的 VARCHAR 长度，对相同数据的查询性能影响很小。

## 当我截断表时出现了“创建分区超时”的错误

截断表时，需要创建相应的分区并进行交换。如果需要创建的分区数量较多，就会出现这个错误。此外，如果有许多数据加载任务，压缩过程中会长时间持有锁，因此在创建表时无法获得锁。如果数据加载任务过多，可以在 **be.conf** 文件中将 `tablet_map_shard_size` 设置为 `512`，以减少锁竞争。

## 访问 Apache Hive 外部表时出现了“无法指定服务器的 Kerberos 主体名称”的错误

在 **fe.conf** 文件和 **be.conf** 文件中添加以下信息到 **hdfs-site.xml** 中：

```HTML
<property>

<name>dfs.namenode.kerberos.principal.pattern</name>

<value>*</value>

</property>
```

## “2021-10” 是 StarRocks 中的日期格式吗？

不是。

## “2021-10” 可以作为分区字段使用吗？

不可以，请使用函数将“2021-10”转换为“2021-10-01”，然后使用“2021-10-01”作为分区字段。

## 在哪里可以查询 StarRocks 数据库或表的大小？

您可以使用[SHOW DATA](../sql-reference/sql-statements/data-manipulation/SHOW_DATA.md)命令。

SHOW DATA；显示当前数据库中所有表的数据大小和副本数。

SHOW DATA FROM <db_name>.<table_name>；显示指定数据库中指定表的数据大小、副本数和行数。
