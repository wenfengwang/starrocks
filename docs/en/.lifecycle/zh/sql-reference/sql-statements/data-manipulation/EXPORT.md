---
displayed_sidebar: English
---

# 导出

## 描述

将表的数据导出到指定位置。

这是一个异步操作。提交导出任务后，将返回导出结果。您可以使用 [SHOW EXPORT](../../../sql-reference/sql-statements/data-manipulation/SHOW_EXPORT.md) 查看导出任务的进度。

> **注意**
> 您只能作为对 StarRocks 表拥有 **EXPORT** 权限的用户，才能从 StarRocks 表中导出数据。如果您没有 **EXPORT** 权限，请按照 [GRANT](../account-management/GRANT.md) 中提供的说明，授予您用于连接 StarRocks 集群的用户 **EXPORT** 权限。

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

  表的名称。StarRocks 支持导出 `engine` 为 `olap` 或 `mysql` 的表数据。

- `partition_name`

  您希望导出数据的分区。默认情况下，如果您未指定此参数，StarRocks 将导出表的所有分区数据。

- `column_name`

  您希望导出数据的列。您使用此参数指定的列顺序可以与表的架构不同。默认情况下，如果您未指定此参数，StarRocks 将导出表的所有列数据。

- `export_path`

  您希望将表数据导出到的位置。如果位置包含路径，请确保路径以斜杠 (/) 结尾。否则，路径中最后一个斜杠 (/) 之后的部分将被用作导出文件名的前缀。默认情况下，如果未指定文件名前缀，则使用 `data_` 作为文件名前缀。

- `opt_properties`

  您可以为导出任务配置的可选属性。

  语法：

  ```SQL
  [PROPERTIES ("<key>"="<value>", ...)]
  ```

  |**属性**|**描述**|
|---|---|
  |column_separator|您希望在导出文件中使用的列分隔符。默认值：`\t`。|
  |line_delimiter|您希望在导出文件中使用的行分隔符。默认值：`\n`。|
  |load_mem_limit|每个单独 BE 上允许的导出任务的最大内存。单位：字节。默认最大内存为 2 GB。|
  |timeout|导出任务超时前的时间长度。单位：秒。默认值：`86400`，即 1 天。|
  |include_query_id|指定导出文件名是否包含 `query_id`。有效值：`true` 和 `false`。`true` 表示文件名包含 `query_id`，`false` 表示文件名不包含 `query_id`。|

- `WITH BROKER`

  在 v2.4 及更早版本中，输入 `WITH BROKER "<broker_name>"` 来指定您想使用的代理。从 v2.5 开始，您不再需要指定代理，但仍需保留 `WITH BROKER` 关键字。更多信息，请参见 [使用 EXPORT 导出数据 > 背景信息](../../../unloading/Export.md#background-information)。

- `broker_properties`

  用于验证源数据的信息。认证信息根据不同的数据源而有所不同。更多信息，请参见 [BROKER LOAD](../../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

## 示例

### 将表的所有数据导出到 HDFS

以下示例将 `testTbl` 表的所有数据导出到 HDFS 集群的 `hdfs://<hdfs_host>:<hdfs_port>/a/b/c/` 路径：

```SQL
EXPORT TABLE testTbl 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/" 
WITH BROKER
(
        "username"="xxx",
        "password"="yyy"
);
```

### 将表的指定分区数据导出到 HDFS

以下示例将 `testTbl` 表的 `p1` 和 `p2` 两个分区的数据导出到 HDFS 集群的 `hdfs://<hdfs_host>:<hdfs_port>/a/b/c/` 路径：

```SQL
EXPORT TABLE testTbl
PARTITION (p1, p2) 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/" 
WITH BROKER
(
        "username"="xxx",
        "password"="yyy"
);
```

### 将表的所有数据导出到 HDFS 并指定列分隔符

以下示例将 `testTbl` 表的所有数据导出到 HDFS 集群的 `hdfs://<hdfs_host>:<hdfs_port>/a/b/c/` 路径，并指定使用逗号（`,`）作为列分隔符：

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

以下示例将 `testTbl` 表的所有数据导出到 HDFS 集群的 `hdfs://<hdfs_host>:<hdfs_port>/a/b/c/` 路径，并指定使用 `\x01`（Hive 默认支持的列分隔符）作为列分隔符：

```SQL
EXPORT TABLE testTbl 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/" 
PROPERTIES
(
    "column_separator"="\\x01"
) 
WITH BROKER;
```

### 将表的所有数据导出到 HDFS 并指定文件名前缀

以下示例将 `testTbl` 表的所有数据导出到 HDFS 集群的 `hdfs://<hdfs_host>:<hdfs_port>/a/b/c/` 路径，并指定使用 `testTbl_` 作为导出文件名的前缀：

```SQL
EXPORT TABLE testTbl 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/testTbl_" 
WITH BROKER;
```

### 将数据导出到 AWS S3

以下示例将 `testTbl` 表的所有数据导出到 AWS S3 存储桶的 `s3a://s3-package/export/` 路径：

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