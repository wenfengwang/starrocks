---
displayed_sidebar: "Chinese"
---

# 导出

## 描述

将表的数据导出到指定位置。

这是一个异步操作。提交导出任务后返回导出结果。您可以使用 [SHOW EXPORT](../../../sql-reference/sql-statements/data-manipulation/SHOW_EXPORT.md) 查看导出任务的进度。

> **注意**
>
> 您只能以具有 StarRocks 表上 EXPORT 权限的用户身份导出 StarRocks 表中的数据。如果您没有 EXPORT 权限，请按照 [GRANT](../account-management/GRANT.md) 中提供的说明，向连接到 StarRocks 集群的用户授予 EXPORT 权限。

## 语法

```SQL
EXPORT TABLE <table_name>
[PARTITION (<partition_name>[, ...])]
[(<column_name>[, ...])]
TO <export_path>
[opt_properties]
WITH BROKER
[broker_properties]
```

## 参数

- `table_name`

  表名称。StarRocks 支持导出 `olap` 或 `mysql` 引擎的表数据。

- `partition_name`

  您要导出数据的分区。默认情况下，如果您不指定此参数，StarRocks 将从表的所有分区导出数据。

- `column_name`

  您要导出数据的列。使用该参数指定的列顺序可以与表模式不同。默认情况下，如果您不指定此参数，StarRocks 将从表的所有列导出数据。

- `export_path`

  您要将表数据导出到的位置。如果位置包含路径，请确保路径以斜杠 (/) 结尾。否则，路径中最后一个斜杠 (/) 后面的部分将用作导出文件名的前缀。如果未指定文件名前缀，则默认使用 `data_` 作为文件名前缀。

- `opt_properties`

  您可以为导出任务配置的可选属性。

  语法:

  ```SQL
  [PROPERTIES ("<key>"="<value>", ...)]
  ```

  | **属性**         | **描述**                                                     |
  | ---------------- | ------------------------------------------------------------ |
  | column_separator | 您要在导出文件中使用的列分隔符。默认值为 `\t`。             |
  | line_delimiter   | 您要在导出文件中使用的行分隔符。默认值为 `\n`。             |
  | load_mem_limit   | 允许在每个单独的 BE 上分配给导出任务的最大内存。单位：字节。默认的最大内存为 2GB。 |
  | timeout          | 导出任务超时时间。单位：秒。默认值是 `86400`，表示 1 天。  |
  | include_query_id | 指定导出文件名称是否包含 `query_id`。有效值：`true` 和 `false`。值 `true` 指定文件名包含 `query_id`，值 `false` 指定文件名不包含 `query_id`。 |

- `WITH BROKER`

  在 v2.4 及更早版本中，输入 `WITH BROKER "<broker_name>"` 以指定要使用的 Broker。从 v2.5 开始，您不再需要指定 Broker，但仍然需要保留 `WITH BROKER` 关键字。有关更多信息，请参阅 [使用 EXPORT 导出数据 > 背景信息](../../../unloading/Export.md#background-information)。

- `broker_properties`

  用于验证源数据的信息。根据数据源不同，认证信息各不相同。有关详细信息，请参阅 [BROKER LOAD](../../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

## 示例

### 将表中所有数据导出到 HDFS

以下示例将 `testTbl` 表的所有数据导出到 HDFS 集群的 `hdfs://<hdfs_host>:<hdfs_port>/a/b/c/` 路径:

```SQL
EXPORT TABLE testTbl 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/" 
WITH BROKER
(
        "username"="xxx",
        "password"="yyy"
);
```

### 将表中指定分区的数据导出到 HDFS

以下示例将 `testTbl` 表的两个分区 `p1` 和 `p2` 的数据导出到 HDFS 集群的 `hdfs://<hdfs_host>:<hdfs_port>/a/b/c/` 路径:

```SQL
EXPORT TABLE testTbl
PARTITION (p1,p2) 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/" 
WITH BROKER
(
        "username"="xxx",
        "password"="yyy"
);
```

### 将表中所有数据导出到 HDFS 并指定列分隔符

以下示例将 `testTbl` 表的所有数据导出到 HDFS 集群的 `hdfs://<hdfs_host>:<hdfs_port>/a/b/c/` 路径，并指定逗号 (`,`) 作为列分隔符:

```SQL
EXPORT TABLE testTbl 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/" 
PROPERTIES
(
    "column_separator"=","
) 
WITH BROKER
(
    "username"="xxx",
    "password"="yyy"
);
```

以下示例将 `testTbl` 表的所有数据导出到 HDFS 集群的 `hdfs://<hdfs_host>:<hdfs_port>/a/b/c/` 路径，并指定 `\x01`（Hive 支持的默认列分隔符）作为列分隔符:

```SQL
EXPORT TABLE testTbl 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/" 
PROPERTIES
(
    "column_separator"="\\x01"
) 
WITH BROKER;
```

### 将表中所有数据导出到 HDFS 并指定文件名前缀

以下示例将 `testTbl` 表的所有数据导出到 HDFS 集群的 `hdfs://<hdfs_host>:<hdfs_port>/a/b/c/testTbl_` 路径，并指定 `testTbl_` 作为导出文件名的前缀:

```SQL
EXPORT TABLE testTbl 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/testTbl_" 
WITH BROKER;
```

### 将数据导出到 AWS S3

以下示例将 `testTbl` 表的所有数据导出到 AWS S3 存储桶的 `s3-package/export/` 路径:

```SQL
EXPORT TABLE testTbl 
TO "s3a://s3-package/export/"
WITH BROKER
(
    "aws.s3.access_key" = "xxx",
    "aws.s3.secret_key" = "yyy",
    "aws.s3.region" = "zzz"
);
```