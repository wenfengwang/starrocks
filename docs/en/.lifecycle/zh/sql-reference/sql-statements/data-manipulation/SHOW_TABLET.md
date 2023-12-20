---
displayed_sidebar: English
---

# 显示 TABLET

## 描述

显示 TABLET 相关信息。

> **注意**
> 对于 v3.0 及更高版本，此操作需要 SYSTEM 级 OPERATE 权限和 TABLE 级 SELECT 权限。对于 v2.5 及更早版本，此操作需要 ADMIN_PRIV 权限。

## 语法

### 查询表或分区中的 TABLET 信息

```sql
SHOW TABLET
FROM [<db_name>.]<table_name>
[PARTITION(<partition_name>, ...)]
[
WHERE [version = <version_number>] 
    [[AND] backendid = <backend_id>] 
    [[AND] STATE = "NORMAL"|"ALTER"|"CLONE"|"DECOMMISSION"]
]
[ORDER BY <field_name> [ASC | DESC]]
[LIMIT [<offset>,]<limit>]
```

|**参数**|**必填**|**描述**|
|---|---|---|
|db_name|否|数据库名称。如果不指定该参数，则默认使用当前数据库。|
|table_name|是|要查询 TABLET 信息的表的名称。必须指定此参数，否则将返回错误。|
|partition_name|否|要查询 TABLET 信息的分区名称。|
|version_number|否|数据版本号。|
|backend_id|否|TABLET 副本所在的 BE 的 ID。|
|STATE|否|TABLET 副本的状态。<ul><li>`NORMAL`：副本正常。</li><li>`ALTER`：正在副本上执行 Rollup 或架构更改。</li><li>`CLONE`：正在克隆副本。（此状态下的副本不可用）。</li><li>`DECOMMISSION`：副本正在退役。</li></ul>|
|field_name|否|结果排序所依据的字段。`SHOW TABLET FROM <table_name>` 返回的所有字段都是可排序的。<ul><li>如果要按升序显示结果，请使用 `ORDER BY field_name ASC`。</li><li>如果要按降序显示结果，请使用 `ORDER BY field_name DESC`。</li></ul>|
|offset|否|要从结果中跳过的 TABLET 数量。例如，`OFFSET 5` 表示跳过前五个 TABLET。默认值：0。|
|limit|否|要返回的 TABLET 数量。例如，`LIMIT 10` 表示仅返回 10 个 TABLET。如果不指定该参数，则返回所有符合过滤条件的 TABLET。|

### 查询单个 TABLET 信息

使用 `SHOW TABLET FROM <table_name>` 获取所有 TABLET ID 后，可以查询单个 TABLET 的信息。

```sql
SHOW TABLET <tablet_id>
```

|**参数**|**必填**|**描述**|
|---|---|---|
|tablet_id|是|TABLET ID|

## 返回字段说明

### 查询表或分区中的 TABLET 信息

```plain
+----------+-----------+-----------+------------+---------+-------------+-------------------+-----------------------+------------------+----------------------+---------------+----------+----------+--------+-------------------------+--------------+------------------+--------------+----------+----------+-------------------+
| TabletId | ReplicaId | BackendId | SchemaHash | Version | VersionHash | LstSuccessVersion | LstSuccessVersionHash | LstFailedVersion | LstFailedVersionHash | LstFailedTime | DataSize | RowCount | State  | LstConsistencyCheckTime | CheckVersion | CheckVersionHash | VersionCount | PathHash | MetaUrl  | CompactionStatus  |
+----------+-----------+-----------+------------+---------+-------------+-------------------+-----------------------+------------------+----------------------+---------------+----------+----------+--------+-------------------------+--------------+------------------+--------------+----------+----------+-------------------+
```

|**字段**|**描述**|
|---|---|
|TabletId|表 ID。|
|ReplicaId|副本 ID。|
|BackendId|副本所在 BE 的 ID。|
|SchemaHash|架构哈希（随机生成）。|
|Version|数据版本号。|
|VersionHash|数据版本号的哈希值。|
|LstSuccessVersion|上次成功加载的版本。|
|LstSuccessVersionHash|上次成功加载版本的哈希值。|
|LstFailedVersion|上次加载失败的版本。`-1` 表示没有版本加载失败。|
|LstFailedVersionHash|上次失败版本的哈希值。|
|LstFailedTime|上次加载失败的时间。`NULL` 表示没有加载失败。|
|DataSize|TABLET 的数据大小。|
|RowCount|TABLET 的数据行数。|
|State|TABLET 的副本状态。|
|LstConsistencyCheckTime|上次一致性检查的时间。`NULL` 表示未执行一致性检查。|
|CheckVersion|执行一致性检查的数据版本。`-1` 表示未检查版本。|
|CheckVersionHash|执行一致性检查的版本的哈希值。|
|VersionCount|数据版本总数。|
|PathHash|TABLET 存储目录的哈希值。|
|MetaUrl|用于查询更多元信息的 URL。|
|CompactionStatus|用于查询数据版本压缩状态的 URL。|

### 查询特定 TABLET 信息

```Plain
+--------+-----------+---------------+-----------+------+---------+-------------+---------+--------+-----------+
| DbName | TableName | PartitionName | IndexName | DbId | TableId | PartitionId | IndexId | IsSync | DetailCmd |
+--------+-----------+---------------+-----------+------+---------+-------------+---------+--------+-----------+
```

|**字段**|**描述**|
|---|---|
|DbName|TABLET 所属数据库的名称。|
|TableName|TABLET 所属表的名称。|
|PartitionName|TABLET 所属分区的名称。|
|IndexName|索引名称。|
|DbId|数据库 ID。|
|TableId|表 ID。|
|PartitionId|分区 ID。|
|IndexId|索引 ID。|
|IsSync|TABLET 上的数据是否与表元一致。`true` 表示数据一致，TABLET 正常。`false` 表示 TABLET 上缺少数据。|
|DetailCmd|用于查询更多信息的 URL。|

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
PARTITION p20210109 VALUES [("2021-01-09"), ("2021-01-10"]))
DISTRIBUTED BY HASH(`k1`, `k2`, `k3`);
```

- 示例 1：查询指定表中所有 TABLET 的信息。以下示例仅从返回信息中摘录了一个 TABLET 的信息。

  ```plain
      mysql> show tablet from example_db.test_show_tablet\G
      *************************** 1. row ***************************
              TabletId: 9588955
              ReplicaId: 9588956
              BackendId: 10004
              SchemaHash: 0
                  Version: 1
              VersionHash: 0
        LstSuccessVersion: 1
    LstSuccessVersionHash: 0
         LstFailedVersion: -1
     LstFailedVersionHash: 0
          LstFailedTime: NULL
              DataSize: 0B
              RowCount: 0
                  State: NORMAL
  LstConsistencyCheckTime: NULL
          CheckVersion: -1
      CheckVersionHash: 0
          VersionCount: 1
              PathHash: 0
               MetaUrl: http://172.26.92.141:8038/api/meta/header/9588955
      CompactionStatus: http://172.26.92.141:8038/api/compaction/show?tablet_id=9588955
  ```

- 示例 2：查询 TABLET 9588955 的信息。

  ```plain
      mysql> show tablet 9588955\G
      *************************** 1. row ***************************
      DbName: example_db
      TableName: test_show_tablet
      PartitionName: p20210103
      IndexName: test_show_tablet
          DbId: 11145
      TableId: 9588953
  PartitionId: 9588946
      IndexId: 9588954
      IsSync: true
      DetailCmd: SHOW PROC '/dbs/11145/9588953/partitions/9588946/9588954/9588955';
  ```

- 示例 3：查询分区 `p20210103` 中的 TABLET 信息。

  ```sql
  SHOW TABLET FROM test_show_tablet PARTITION(p20210103);
  ```

- 示例 4：返回 10 个 TABLET 的信息。

  ```sql
      SHOW TABLET FROM test_show_tablet LIMIT 10;
  ```

- 示例 5：返回偏移量为 5 的 10 个 TABLET 的信息。

  ```sql
  SHOW TABLET FROM test_show_tablet LIMIT 5,10;
  ```

- 示例 6：按 `backend_id`、`version` 和 `state` 过滤 TABLET。

  ```sql
      SHOW TABLET FROM test_show_tablet
      WHERE backend_id = 10004 AND version = 1 AND state = "NORMAL";
  ```

- 示例 7：按 `version` 对 TABLET 进行排序。

  ```sql
      SHOW TABLET FROM test_show_tablet WHERE backend_id = 10004 ORDER BY version;
  ```

- 示例 8：返回索引名为 `test_show_tablet` 的 TABLET 信息。

  ```sql
  SHOW TABLET FROM test_show_tablet WHERE index_name = "test_show_tablet";
  ```