---
displayed_sidebar: "Chinese"
---

# 显示分区

## 描述

显示分区信息，包括常规分区和[临时分区](../../../table_design/Temporary_partition.md)。

## 语法

```sql
SHOW [TEMPORARY] PARTITIONS FROM [db_name.]table_name [WHERE] [ORDER BY] [LIMIT]
```

> 注意
>
> 该语法仅支持StarRocks表（`"ENGINE" = "OLAP"`）。
> 从v3.0开始，此操作需要在指定表上具有SELECT权限。对于v2.5及更早版本，此操作需要在指定表上具有SELECT__PRIV权限。

## 返回字段描述

```plaintext
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+-------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
| PartitionId | PartitionName | VisibleVersion | VisibleVersionTime  | VisibleVersionHash | State  | PartitionKey | Range | DistributionKey    | Buckets | ReplicationNum | StorageMedium | CooldownTime        | LastConsistencyCheckTime | DataSize | IsInMemory | RowCount |
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+-------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
```

| **字段**         | **描述**                                                 |
| ---------------- | ------------------------------------------------------- |
| PartitionId      | 分区的ID。                                   |
| PartitionName    | 分区的名称。                                        |
| VisibleVersion   | 最后一个成功加载事务的版本号。每次成功加载事务后，版本号增加1。 |
| VisibleVersionTime | 最后一个成功加载事务的时间戳。                           |
| VisibleVersionHash | 最后一个成功加载事务的版本号的哈希值。                   |
| State            | 分区的状态。固定值为`Normal`。                              |
| PartitionKey     | 由一个或多个分区列组成的分区键。                         |
| Range            | 分区的范围，是右半开区间。                                   |
| DistributionKey  | 散列分桶的桶键。                                         |
| Buckets          | 分区的桶数。                                           |
| ReplicationNum   | 分区中每个tablet的副本数。                            |
| StorageMedium    | 用于存储分区数据的存储介质。值`HHD`表示硬盘驱动器，值`SSD`表示固态硬盘驱动器。 |
| CooldownTime     | 分区数据的冷却时间。如果初始存储介质为SSD，那么在指定参数时间后，存储介质从SSD切换到HDD。格式："yyyy-MM-dd HH:mm:ss"。 |
| LastConsistencyCheckTime | 最后一次一致性检查的时间。`NULL`表示未执行一致性检查。    |
| DataSize         | 分区中数据的大小。                                      |
| IsInMemory       | 分区中所有数据是否存储在内存中。                          |
| RowCount         | 分区的数据行数。                                       |

## 示例

1. 显示指定数据库`test`下指定表`site_access`的所有常规分区信息。

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

2. 显示指定数据库`test`下指定表`site_access`的所有临时分区信息。

    ```sql
    SHOW TEMPORARY PARTITIONS FROM test.site_access;
    ```

3. 显示指定数据库`test`下指定表`site_access`的指定分区`p1`的信息。

    ```sql
    -- 常规分区
    SHOW PARTITIONS FROM test.site_access WHERE PartitionName = "p1";
    -- 临时分区
    SHOW TEMPORARY PARTITIONS FROM test.site_access WHERE PartitionName = "p1";
    ```

4. 显示指定数据库`test`下指定表`site_access`的最新分区信息。

    ```sql
    -- 常规分区
    SHOW PARTITIONS FROM test.site_access ORDER BY PartitionId DESC LIMIT 1;
    -- 临时分区
    SHOW TEMPORARY PARTITIONS FROM test.site_access ORDER BY PartitionId DESC LIMIT 1;
    ```