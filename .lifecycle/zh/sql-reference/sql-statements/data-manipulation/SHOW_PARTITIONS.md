---
displayed_sidebar: English
---

# 显示分区信息

## 描述

展示包括普通分区和 [临时分区](../../../table_design/Temporary_partition.md) 在内的分区信息。

## 语法说明

```sql
SHOW [TEMPORARY] PARTITIONS FROM [db_name.]table_name [WHERE] [ORDER BY] [LIMIT]
```

> 注意
> 此语法仅支持 StarRocks 表（"ENGINE" = "OLAP"）。从 v3.0 版本开始，执行此操作需要对指定表拥有 SELECT 权限。对于 v2.5 及更早版本，执行此操作需要对指定表拥有 SELECT__PRIV 权限。

## 返回字段描述

```plaintext
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+-------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
| PartitionId | PartitionName | VisibleVersion | VisibleVersionTime  | VisibleVersionHash | State  | PartitionKey | Range | DistributionKey    | Buckets | ReplicationNum | StorageMedium | CooldownTime        | LastConsistencyCheckTime | DataSize | IsInMemory | RowCount |
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+-------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
```

|字段|描述|
|---|---|
|PartitionId|分区的 ID。|
|PartitionName|分区的名称。|
|VisibleVersion|上次成功加载事务的版本号。每次成功加载事务后，版本号都会增加 1。|
|VisibleVersionTime|上次成功加载事务的时间戳。|
|VisibleVersionHash|上次成功加载事务的版本号的哈希值。|
|状态|分区的状态。固定值：正常。|
|PartitionKey|由一个或多个分区列组成的分区键。|
|范围|分区的范围，为右半开区间。|
|DistributionKey|哈希分桶的桶键。|
|Buckets|分区的存储桶数量。|
|ReplicationNum|分区中每个tablet 的副本数量。|
|StorageMedium|分区中存储数据的存储介质。值 HHD 表示硬盘驱动器，值 SSD 表示固态驱动器。|
|CooldownTime|分区中数据的冷却时间。如果初始存储介质为SSD，则经过该参数指定的时间后，存储介质从SSD切换为HDD。格式：“yyyy-MM-dd HH:mm:ss”。|
|LastConsistencyCheckTime|上次一致性检查的时间。 NULL 表示未执行一致性检查。|
|DataSize|分区中数据的大小。|
|IsInMemory|分区中的所有数据是否都存储在内存中。|
|RowCount|分区的数据行数。|
|MaxCS|分区的最大压缩分数。仅适用于共享数据集群。|

## 示例

1. 展示指定数据库 test 下指定表 site_access 的所有常规分区信息。

   ```SQL
   MySQL > show partitions from test.site_access\G
   *************************** 1. row ***************************
               PartitionId: 20990
           PartitionName: p2019 
           VisibleVersion: 1
       VisibleVersionTime: 2023-08-08 15:45:13
       VisibleVersionHash: 0
                   State: NORMAL
               PartitionKey: datekey
                   Range: [types: [DATE]; keys: [2019-01-01]; ..types: [DATE]; keys: [2020-01-01]; )
           DistributionKey: site_id
                   Buckets: 6
           ReplicationNum: 3
           StorageMedium: HDD
               CooldownTime: 9999-12-31 23:59:59
   LastConsistencyCheckTime: NULL
                   DataSize:  4KB   
               IsInMemory: false
                   RowCount: 3 
   1 row in set (0.00 sec)
   ```

2. 展示指定数据库 test 下指定表 site_access 的所有临时分区信息。

   ```sql
   SHOW TEMPORARY PARTITIONS FROM test.site_access;
   ```

3. 展示指定数据库 test 下指定表 site_access 的特定分区 p1 的信息。

   ```sql
   -- Regular partition
   SHOW PARTITIONS FROM test.site_access WHERE PartitionName = "p1";
   -- Temporary partition
   SHOW TEMPORARY PARTITIONS FROM test.site_access WHERE PartitionName = "p1";
   ```

4. 展示指定数据库 test 下指定表 site_access 的最新分区信息。

   ```sql
   -- Regular partition
   SHOW PARTITIONS FROM test.site_access ORDER BY PartitionId DESC LIMIT 1;
   -- Temporary partition
   SHOW TEMPORARY PARTITIONS FROM test.site_access ORDER BY PartitionId DESC LIMIT 1;
   ```
