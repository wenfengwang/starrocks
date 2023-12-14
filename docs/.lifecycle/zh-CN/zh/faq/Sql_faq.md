```yaml
displayed_sidebar: "Chinese"
```

# 查询相关问题

## 构建物化视图失败：fail to allocate memory

修改 `be.conf` 中的`memory_limitation_per_thread_for_schema_change`。

该参数表示单个 schema change 任务允许占用的最大内存，默认大小 2G。修改完成后，需重启 BE 使配置生效。

## StarRocks 会缓存查询结果吗？

StarRocks 不直接缓存最终查询结果。从 2.5 版本开始，StarRocks 会将多阶段聚合查询的第一阶段聚合的中间结果缓存在 Query Cache 里，后续查询可以复用之前缓存的结果，加速计算。Query Cache 占用所在 BE 的内存。更多信息，参见 [Query Cache](../using_starrocks/query_cache.md)。


## 当字段为NULL时，除了is null， 其他所有的计算结果都是false

标准 SQL 中 null 和其他表达式计算结果都是null。

## [BIGINT 等值查询中加引号] 出现多余数据

```sql
select cust_id,idno 
from llyt_dev.dwd_mbr_custinfo_dd 
where Pt= ‘2021-06-30’ 
and cust_id = ‘20210129005809043707’ 
limit 10 offset 0;
```

```plain text
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
```

```sql
select cust_id,idno 
from llyt_dev.dwd_mbr_custinfo_dd 
where Pt= ‘2021-06-30’ 
and cust_id = 20210129005809043707 
limit 10 offset 0;
```

```plain text
+---------------------+-----------------------------------------+
|   cust_id           |      idno                               |
+---------------------+-----------------------------------------+
|  20210189979989976  | xuywehuhfuhruehfurhghcfCNMX39dw=        |
+---------------------+-----------------------------------------+
```

**问题描述**

WHERE 里使用 BIGINT 类型，查询加单引号，查出很多无关数据。

**解决方案**

字符串和 INT 比较，相当于 CAST 成 DOUBLE。INT 比较时，不要加引号。加了引号，还会导致无法命中索引。

## StarRocks有decode函数吗？

StarRocks 不支持 Oracle 中的 decode 函数，StarRocks 语法兼容 MySQL，可以使用case when。

## StarRocks的主键覆盖是立刻生效的吗？还是说要等后台慢慢合并数据?

StarRocks 的后台合并参考 Google 的 MESA 模型，有两层 compaction，会后台策略触发合并。如果没有合并完成，查询时会合并，但是读出来只会有一个最新的版本，不存在「导入后数据读不到最新版本」的情况。

## StarRocks 存储 utf8mb4 的字符，会不会被截断或者乱码？

MySQL的“utf8mb4”是标准的“UTF-8”，StarRocks 可以完全兼容。

## [Schema change] alter table 时显示：table's state is not normal

ALTER TABLE 是异步操作，如果之前有 ALTER TABLE 操作还没完成，可以通过如下语句查看 ALTER 状态。

```sql
show tablet from lineitem where State="ALTER"; 
```

执行时间与数据量大小有关系，一般是分钟级别，建议 ALTER 过程中停止数据导入，导入会降低 ALTER 速度。

## [Hive外部表查询问题] 查询 Hive 外部表时报错获取分区失败

**问题描述**

查询 Hive 外部表时具体报错信息：
`get partition detail failed: com.starrocks.common.DdlException: get hive partition meta data failed: java.net.UnknownHostException:hadooptest（具体hdfs-ha的名字）`

**解决方案**

将`core-site.xml`和`hdfs-site.xml`文件拷贝到 `fe/conf` 和 `be/conf`中即可，然后重启 FE 和 BE。

**问题原因**

获取配置单元分区元数据失败。

## 大表查询结果慢，没有谓词下推

多张大表关联时，旧 planner有时没有自动谓词下推，比如：

```sql
A JOIN B ON A.col1=B.col1 JOIN C on B.col1=C.col1 where A.col1='北京'，
```

可以更改为：

```sql
A JOIN B ON A.col1=B.col1 JOIN C on A.col1=C.col1 where A.col1='北京'，
```

或者升级到较新版本并开启 CBO 功能，会有此类谓词下推操作，优化查询性能。

## 查询报错 planner use long time 3000 remaining task num 1

**解决方案**

查看`fe.gc`日志确认该问题是否是多并发引起的full gc。

如果查看后台监控和日志初步判断有频繁gc，可参考两个方案：

  1. 让sqlclient同时访问多个FE去做负载均衡。
  2. 修改`fe.conf`中 `jvm8g` 为16g（更大内存，减少 full gc 影响）。修改后需重启FE。

## 当A基数很小时，select B from tbl order by A limit 10查询结果每次不一样

**解决方案：**

使用`select B from tbl order by A,B limit 10` ，将B也进行排序就能保证结果一致。

**问题原因**

上面的SQL只能保证A是有序的，并不能保证每次查询出来的B顺序是一致的，MySQL能保证这点因为它是单机数据库，而StarRocks是分布式数据库，底层表数据存储是sharding的。A的数据分布在多台机器上，每次查询多台机器返回的顺序可能不同，就会导致每次B顺序不一致。

## select * 和 select 的列效率差距过大

select * 和 select 时具体列效率差距会很大，这时应该去排查profile，看 MERGE 的具体信息。

* 确认是否是存储层聚合消耗的时间过长。
* 确认是否指标列有很多，需要对几百万行的几百列进行聚合。

```plain text
MERGE:
    - aggr: 26s270ms
    - sort: 15s551ms
```

## Currently, nested functions are not supported in DELETE

Currently, nested operations like the following are not supported: `DELETE from test_new WHERE to_days(now())-to_days(publish_time) >7;`. Here 'to_days(now())' is nested.

## If there are hundreds of tables in a database, the use database operation may be very slow

When the client connects, add the `-A` parameter, for example: `mysql -uroot -h127.0.0.1 -P8867 -A`. `-A` skips pre-reading database information, and switching databases will be faster.

## Too many BE and FE log files, how to deal with them?

Adjust the log level and parameter size. For details, refer to the default values and explanations of log-related parameters: [Parameter Configuration](../administration/Configuration.md).

## Failed to change the number of replicas: table lineorder is colocate table, cannot change replicationNum

Colocate tables belong to a group. A group contains multiple tables, and it does not support modifying the number of replicas for individual tables. You need to set the `group_with` attribute of all tables inside the group to empty, then set the `replication_num` for all tables, and finally set the `group_with` attribute back.

## Does setting varchar to the maximum value have any impact on storage?

VARCHAR is a variable-length storage, and storage depends on the actual length of the data. Specifying different VARCHAR lengths when creating a table has little impact on the query performance of the same data.

## Failed to truncate table, error reported as create partititon timeout

Currently, TRUNCATE first creates the corresponding empty partition and then swaps. If there are a large number of partition creation tasks in backlog, it will time out. The compaction process will hold the lock for a long time, which also causes the table creation to not acquire the lock. If the cluster has a large import rate, setting the parameter `tablet_map_shard_size=512` in `be.conf` can reduce lock conflicts. After modifying the parameter, you need to restart the FE.

## Error accessing Hive external table, Failed to specify server's Kerberos principal name

Add the following information to `fe.conf` and `be.conf` in `hdfs-site.xml`:

```plain text
<property>
<name>dfs.namenode.kerberos.principal.pattern</name>
<value>*</value>
</property>
```

## Is "2021-10" a valid date format in StarRocks? Can it be used as a partition field?

It is not a valid date format and cannot be used as a partition field. It needs to be adjusted to `2021-10-01` before partitioning.

## When creating an Elasticsearch external table in StarRocks, if the related string length exceeds 256, and Elasticsearch uses dynamic mapping, selecting from the table will result in the column not being queried

During dynamic mapping, the data type in Elasticsearch is as follows:

```json
          "k4": {
                "type": "text",
                "fields": {
                   "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                   }
                }
             }
```

StarRocks uses the keyword data type to transform this query statement. Because the length of keyword data in this column exceeds 256, the column cannot be queried.

Solution: Remove the following from the field mapping:

```json
            "fields": {
                   "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                   }
                }
```

Using the text type will resolve this issue.

## How to quickly estimate the size of StarRocks databases and tables, and the disk resources they occupy?

You can use the [SHOW DATA](../sql-reference/sql-statements/data-manipulation/SHOW_DATA.md) command to view the storage size of databases and tables.

`SHOW DATA;` displays the data size and number of replicas for all tables in the current database.

`SHOW DATA FROM <db_name>.<table_name>;` shows the data size, number of replicas, and statistical number of rows for a specific table in the specified database.