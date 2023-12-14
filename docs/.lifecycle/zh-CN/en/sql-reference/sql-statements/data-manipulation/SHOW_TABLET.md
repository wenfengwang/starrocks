---
displayed_sidebar: "Chinese"
---

# 显示 TABLET

## 描述

显示与tablet相关的信息。

> **注意**
>
> 对于 v3.0 及更高版本，此操作需要SYSTEM级别的操作权限和TABLE级别的SELECT权限。对于 v2.5 及更早版本，此操作需要ADMIN_PRIV权限。

## 语法

### 查询表或分区中的tablets信息

```sql
SHOW TABLET
FROM [<db_name>.]<table_name>
[PARTITION(<partition_name>, ...]
[
WHERE [version = <version_number>] 
    [[AND] backendid = <backend_id>] 
    [[AND] STATE = "NORMAL"|"ALTER"|"CLONE"|"DECOMMISSION"]
]
[ORDER BY <field_name> [ASC | DESC]]
[LIMIT [<offset>,]<limit>]
```

| **参数**       | **是否必需** | **描述**                                                     |
| -------------- | -------- | ------------------------------------------------------------ |
| db_name        | 否       | 数据库名称。如果不指定此参数，将默认使用当前数据库。                     |
| table_name     | 是       | 要查询tablet信息的表的名称。必须指定此参数，否则会返回错误。                     |
| partition_name | 否       | 要查询tablet信息的分区的名称。                                                     |
| version_number | 否       | 数据版本号。                                                   |
| backend_id     | 否       | 定位tablet副本的BE的ID。                                 |
| STATE          | 否       | tablet副本的状态。<ul><li>`NORMAL`: 副本正常。</li><li>`ALTER`: 副本正在进行Rollup或模式更改。</li><li>`CLONE`: 副本正在克隆中。（此状态下的副本不可用）。 </li><li>`DECOMMISSION`: 副本正在被弃用。 </li></ul> |
| field_name     | 否       | 按照哪个字段排序结果。`SHOW TABLET FROM <table_name>`返回的所有字段均可排序。<ul><li>如果希望按升序显示结果，请使用 `ORDER BY field_name ASC`。</li><li>如果希望按降序显示结果，请使用 `ORDER BY field_name DESC`。</li></ul> |
| offset         | 否       | 跳过结果中的tablet数量。例如，`OFFSET 5` 表示跳过前5个tablet。默认值：0。 |
| limit          | 否       | 要返回的tablet数量。例如， `LIMIT 10` 表示仅返回10个tablet。如果未指定此参数，则返回满足过滤条件的所有tablet。 |

### 查询特定tablet的信息

在使用 `SHOW TABLET FROM <table_name>` 获取所有tablet ID 后，可以查询单个tablet的信息。

```sql
SHOW TABLET <tablet_id>
```

| **参数**  | **是否必需** | **描述**       |
| --------- | -------- | -------------- |
| tablet_id | 是       | tablet ID |

## 返回字段的描述

### 查询表或分区中的tablets信息

```plain
+----------+-----------+-----------+------------+---------+-------------+-------------------+-----------------------+------------------+----------------------+---------------+----------+----------+--------+-------------------------+--------------+------------------+--------------+----------+----------+-------------------+
| TabletId | ReplicaId | BackendId | SchemaHash | Version | VersionHash | LstSuccessVersion | LstSuccessVersionHash | LstFailedVersion | LstFailedVersionHash | LstFailedTime | DataSize | RowCount | State  | LstConsistencyCheckTime | CheckVersion | CheckVersionHash | VersionCount | PathHash | MetaUrl  | CompactionStatus  |
+----------+-----------+-----------+------------+---------+-------------+-------------------+-----------------------+------------------+----------------------+---------------+----------+----------+--------+-------------------------+--------------+------------------+--------------+----------+----------+-------------------+
```

| **字段**                | **描述**                        |
| ----------------------- | ------------------------------- |
| TabletId                | table ID。                   |
| ReplicaId               | 副本ID。                      |
| BackendId               | 副本所在BE的ID。  |
| SchemaHash              | 模式哈希（随机生成）。        |
| Version                 | 数据版本号。                     |
| VersionHash             | 数据版本号的哈希。              |
| LstSuccessVersion       | 最后成功加载的版本。        |
| LstSuccessVersionHash   | 最后成功加载版本的哈希。 |
| LstFailedVersion        | 最后失败加载的版本。

`-1` 表示没有加载失败的版本。 |
| LstFailedVersionHash    | 最后失败版本的哈希。 |
| LstFailedTime           | 最后失败加载的时间。 `NULL` 表示没有加载失败。|
| DataSize                | tablet的数据大小。          |
| RowCount                | tablet的数据行数。            |
| State                   | tablet的副本状态。           |
| LstConsistencyCheckTime | 最后一次一致性检查的时间。 `NULL` 表示未执行一致性检查。 |
| CheckVersion            | 进行一致性检查的数据版本。`-1` 表示没有检查过版本。    |
| CheckVersionHash        | 进行一致性检查的版本的哈希。         |
| VersionCount            | 数据版本的总数。                      |
| PathHash                | 存储tablet的目录的哈希。        |
| MetaUrl                 | 查询更多元信息的URL。     |
| CompactionStatus        | 查询数据版本压缩状态的URL。    |

### 查询特定tablet的信息

```Plain
+--------+-----------+---------------+-----------+------+---------+-------------+---------+--------+-----------+
| DbName | TableName | PartitionName | IndexName | DbId | TableId | PartitionId | IndexId | IsSync | DetailCmd |
+--------+-----------+---------------+-----------+------+---------+-------------+---------+--------+-----------+
```

| **字段**      | **描述**                                                     |
| ------------- | ------------------------------------------------------------ |
| DbName        | 此tablet所属的数据库名称。      |
| TableName     | 此tablet所属的表名称。         |
| PartitionName | 此tablet所属的分区名称。     |
| IndexName     | 索引名称。                                          |
| DbId          | 数据库ID。                                           |
| TableId       | 表ID。                                              |
| PartitionId   | 分区ID。                                          |
| IndexId       | 索引ID。                                              |
| IsSync        | tablet上的数据是否与表元数据一致。`true` 表示数据一致，tablet 正常。 `false` 表示tablet上的数据缺失。 |
| DetailCmd     | 查询更多信息的URL。                    |

## 示例

在数据库 `example_db` 中创建表 `test_show_tablet`。

```sql
CREATE TABLE `test_show_tablet` (
  `k1` date NULL COMMENT "",
  `k2` datetime NULL COMMENT "",
  `k3` char(20) NULL COMMENT "",
  `k4` varchar(20) NULL COMMENT "",
  `k5` boolean NULL COMMENT "",
  `k6` tinyint(4) NULL COMMENT "",
  `k7` smallint(6) NULL COMMENT "",
  `k8` int(11) NULL COMMENT "",
  `k9` bigint(20) NULL COMMENT "",
  `k10` largeint(40) NULL COMMENT "",
  `k11` float NULL COMMENT "",
  `k12` double NULL COMMENT "",
  `k13` decimal128(27, 9) NULL COMMENT ""
) ENGINE=OLAP 
DUPLICATE KEY(`k1`, `k2`, `k3`, `k4`, `k5`)
COMMENT "OLAP"
PARTITION BY RANGE(`k1`)
(PARTITION p20210101 VALUES [("2021-01-01"), ("2021-01-02")),
PARTITION p20210102 VALUES [("2021-01-02"), ("2021-01-03")),
PARTITION p20210103 VALUES [("2021-01-03"), ("2021-01-04")),
PARTITION p20210104 VALUES [("2021-01-04"), ("2021-01-05")),
PARTITION p20210105 VALUES [("2021-01-05"), ("2021-01-06")),
PARTITION p20210106 VALUES [("2021-01-06"), ("2021-01-07")),
PARTITION p20210107 VALUES [("2021-01-07"), ("2021-01-08")),
PARTITION p20210108 VALUES [("2021-01-08"), ("2021-01-09")),
PARTITION p20210109 VALUES [("2021-01-09"), ("2021-01-10")))
DISTRIBUTED BY HASH(`k1`, `k2`, `k3`);
```

- 示例 1：查询指定表中所有tablet的信息。下面的示例从返回信息中摘取了一个tablet的信息。

    ```plain
        mysql> show tablet from example_db.test_show_tablet\G
        *************************** 1. row ***************************
                TabletId: 9588955
```
                副本Id: 9588956
                后端Id: 10004
                模式哈希: 0
                    版本: 1
                版本哈希: 0
          最近成功版本: 1
      最近成功版本哈希: 0
           最近失败版本: -1
       最近失败版本哈希: 0
            最近失败时间: NULL
                数据大小: 0B
                行数: 0
                    状态: 正常
    最近一致性检查时间: NULL
            检查版本: -1
        检查版本哈希: 0
            版本计数: 1
                路径哈希: 0
                 元信息网址: http://172.26.92.141:8038/api/meta/header/9588955
        压实状态: http://172.26.92.141:8038/api/compaction/show?tablet_id=9588955
    ```

- 示例2: 查询表格9588955的信息。

    ```plain
        mysql> show tablet 9588955\G
        *************************** 1. row ***************************
        数据库名称: example_db
        表名称: test_show_tablet
        分区名称: p20210103
        索引名称: test_show_tablet
            数据库Id: 11145
        表格Id: 9588953
    分区Id: 9588946
        索引Id: 9588954
        是否同步: true
        详细命令: SHOW PROC '/dbs/11145/9588953/partitions/9588946/9588954/9588955';
    ```

- 示例3: 查询`p20210103`分区内的表格信息。

    ```sql
    SHOW TABLET FROM test_show_tablet partition(p20210103);
    ```

- 示例4: 返回10个表格的信息。

    ```sql
        SHOW TABLET FROM test_show_tablet limit 10;
    ```

- 示例5: 带有偏移量5的10个表格的信息。

    ```sql
    SHOW TABLET FROM test_show_tablet limit 5,10;
    ```

- 示例6: 通过`backendid`、`version`和`state`筛选表格。

    ```sql
        SHOW TABLET FROM test_show_tablet
        WHERE backendid = 10004 and version = 1 and state = "NORMAL";
    ```

- 示例7: 按`version`对表格排序。

    ```sql
        SHOW TABLET FROM table_name where backendid = 10004 order by version;
    ```

- 示例8: 返回索引名为`test_show_tablet`的表格信息。

    ```sql
    SHOW TABLET FROM test_show_tablet where indexname = "test_show_tablet";
    ```