---
displayed_sidebar: "Chinese"
---

# SQL

本主题提供有关 SQL 的一些常见问题的答案。

## 当我构建材料化视图时出现"分配内存失败。"的错误

要解决此问题，请增加**be.conf**文件中`memory_limitation_per_thread_for_schema_change`参数的值。该参数是指用于更改方案的单个任务可以分配的最大存储空间。最大存储空间的默认值为 2 GB。

## StarRocks 是否支持缓存查询结果？

StarRocks 不直接缓存最终的查询结果。从 v2.5 开始，StarRocks 使用查询缓存功能将第一阶段聚合的中间结果保存到缓存中。与之前的查询语义相同的新查询可以重用缓存的计算结果以加速计算。查询缓存使用 BE 存储器。有关更多信息，请参见 [查询缓存](../using_starrocks/query_cache.md)。

## 当计算中包含`Null`时，除 ISNULL() 函数外其他函数的计算结果均为false

在标准 SQL 中，每个包含具有`NULL`值的操作数的计算结果都会返回`NULL`。

## 将 BIGINT 数据类型的值用引号括起来进行等值查询时，为什么查询结果不正确？

### 问题描述

请参阅以下示例：

```plaintext
select 编号,身份证号

from llyt_dev.dwd_mbr_custinfo_dd 

where 时间= ‘2021-06-30’ 

and 编号 = ‘20210129005809043707’ 

limit 10 offset 0;
+---------------------+-----------------------------------------+

|   编号             |      身份证号                            |

+---------------------+-----------------------------------------+

|  20210129005809436  | yjdgjwsnfmdhjw294F93kmHCNMX39dw=        |

|  20210129005809436  | sdhnswjwijeifme3kmHCNMX39gfgrdw=        |

|  20210129005809436  | Tjoedk3js82nswndrf43X39hbggggbw=        |

|  20210129005809436  | denuwjaxh73e39592jwshbnjdi22ogw=        |

|  20210129005809436  | ckxwmsd2mei3nrunjrihj93dm3ijin2=        |

|  20210129005809436  | djm2emdi3mfi3mfu4jro2ji2ndimi3n=        |

+---------------------+-----------------------------------------+
select 编号, 身份证号

from llyt_dev.dwd_mbr_custinfo_dd 

where 时间= ‘2021-06-30’ 

and 编号 = 20210129005809043707 

limit 10 offset 0;
+---------------------+-----------------------------------------+

|   编号             |      身份证号                            |

+---------------------+-----------------------------------------+

|  20210189979989976  | xuywehuhfuhruehfurhghcfCNMX39dw=        |

+---------------------+-----------------------------------------+
```

### 解决方法

当您比较 STRING 数据类型和 INTEGER 数据类型时，这两种类型的字段将转换为 DOUBLE 数据类型。因此，不能添加引号。否则，WHERE 子句中定义的条件将无法进行索引。

## StarRocks 是否支持 DECODE 函数？

StarRocks 不支持 Oracle 数据库的 DECODE 函数。StarRocks 与 MySQL 兼容，因此您可以使用 CASE WHEN 语句。

## 在将数据加载到 StarRocks 的主键表后，是否可以立即查询最新数据？

可以。StarRocks 合并数据的方式参考了 Google Mesa。在 StarRocks 中，BE 触发数据合并，并且具有两种压缩方式以合并数据。如果数据合并未完成，在执行查询时将完成数据合并。因此，在数据加载后，您可以读取最新数据。

## 存储在 StarRocks 中的 utf8mb4 字符是否会被截断或显示为乱码？

不会。

## 运行`alter table`命令时出现“表的状态不正常”错误

此错误是因为之前的修改未完成。您可以运行以下代码来检查之前修改的状态：

```SQL
show tablet from lineitem where State="ALTER"; 
```

改动操作所需的时间与数据量有关。一般情况下，改动可在几分钟内完成。我们建议您在进行表更改时停止向 StarRocks 加载数据，因为数据加载会降低更改速度。

## 查询 Apache Hive 的外部表时出现“获取分区明细失败：org.apache.doris.common.DdlException: 获取 Hive 分区元数据失败：java.net.UnknownHostException:hadooptest”错误

此错误是因为无法获取 Apache Hive 分区的元数据。要解决此问题，请将**core-sit.xml**和**hdfs-site.xml**复制到**fe.conf**文件和**be.conf**文件中。

## 查询数据时出现“planner use long time 3000 remaining task num 1”错误

通常，此错误是由于完整垃圾收集（full GC）引起的，可以通过后端监控和**fe.gc**日志进行检查。要解决此问题，请执行以下操作之一：

- 允许 SQL 客户端同时访问多个前端 (FEs) 以分散负载。
- 在**fe.conf**文件中，将 Java 虚拟机 (JVM) 的堆大小从 8 GB 更改为 16 GB 以增加内存并减少完整 GC 的影响。

## 当列 A 的基数较小时，使用`select B from tbl order by A limit 10`的查询结果每次都不同

SQL 只能保证列 A 有序，无法保证列 B 的顺序每次都相同的查询结果。MySQL 可以保证列 A 和列 B的顺序, 因为它是独立的数据库。

StarRocks 是一个分布式数据库，底层表中存储的数据是分片模式的。列 A 的数据分布在多台机器上，因此，多台机器返回的列 B 的顺序可能会因每次查询而不同，导致每次 B 的顺序不一致。要解决此问题，将`select B from tbl order by A limit 10` 改为 `select B from tbl order by A,B limit 10`。

## SELECT * 和 SELECT 之间的列效率差距为何如此之大？

要解决此问题，请检查配置文件并查看MERGE详细信息：

- 检查存储层的聚合是否花费了太多时间。

- 检查是否有太多的指标列。如果有太多指标列，可能会对数百万行的数据进行聚合。

```plaintext
MERGE:

    - aggr: 26s270ms

    - sort: 15s551ms
```

## DELETE 支持嵌套函数吗？

不支持嵌套函数，例如在`DELETE from test_new WHERE to_days(now())-to_days(publish_time) >7;`中的`to_days(now())`。

## 当数据库中有数百张表时，如何提高数据库的使用效率？

要提高效率，在连接到 MySQL 客户端服务器时添加`-A`参数：`mysql -uroot -h127.0.0.1 -P8867 -A`。MySQL 客户端服务器不预读数据库信息。

## 如何减少 BE 日志和 FE 日志占用的磁盘空间？

调整日志级别和相应参数。有关更多信息，请参见[参数配置](../administration/Configuration.md)。

## 修改复制数时出现“表 *** 是 colocate 表，无法更改复制数”错误

创建 colocate 表时，需要设置`group`属性。因此，不能为单个表修改复制数。您可以执行以下步骤来修改组中所有表的复制数：

1. 为组中的所有表设置`group_with`为`empty`。
2. 为组中的所有表设置适当的`replication_num`。
3. 将`group_with`设置回其原始值。

## 将 VARCHAR 设置为最大值会影响存储吗？

VARCHAR 是可变长度数据类型，其指定长度可根据实际数据长度进行更改。在创建表时指定不同的 varchar 长度对相同数据的查询性能影响较小。

## 截断表时出现“create partititon timeout”错误

要截断表，您需要创建相应的分区，然后进行交换。如果需要创建的分区过多，将会出现此错误。此外，如果存在许多数据加载任务，那么在压缩过程中将持有锁很长时间。因此，在创建表时无法获取锁定。如果存在过多的数据加载任务，在**be.conf**文件中将`tablet_map_shard_size`设置为`512`以减少锁争用。

## 查询 Apache Hive 的外部表时出现“Failed to specify server's Kerberos principal name”错误

将以下信息添加到**hdfs-site.xml**文件的**fe.conf**文件和**be.conf**文件中：

```HTML
<property>

<name>dfs.namenode.kerberos.principal.pattern</name>

<value>*</value>

</property>
```

## "2021-10" 是 StarRocks 中的日期格式吗？

不是。

## "2021-10" 可以作为分区字段使用吗？

不可以，需要使用函数将"2021-10"更改为"2021-10-01"，然后将"2021-10-01"作为分区字段使用。

## 我在哪里可以查询 StarRocks 数据库或表的大小？

您可以使用[SHOW DATA](../sql-reference/sql-statements/data-manipulation/SHOW_DATA.md)命令。

`SHOW DATA;` 显示当前数据库中所有表的数据大小和副本情况。

`SHOW DATA FROM <db_name>.<table_name>;` 显示指定数据库中指定表的数据大小、副本数和行数。