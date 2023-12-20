---
displayed_sidebar: English
---

# 导出

## 描述

将表的数据导出到指定位置。

这是一个异步操作。提交导出任务后，将返回导出结果。您可以使用 [SHOW EXPORT](../../../sql-reference/sql-statements/data-manipulation/SHOW_EXPORT.md) 命令查看导出任务的进度。

> **注意事项**
> 只有对 StarRocks 表拥有 **EXPORT** 权限的用户才能从 StarRocks 表中导出数据。如果您没有 **EXPORT** 权限，请按照 [GRANT](../account-management/GRANT.md) 命令中提供的说明，将 **EXPORT** 权限授予用于连接 StarRocks 集群的用户。

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

- table_name

  表的名称。StarRocks 支持导出引擎为 OLAP 或 MySQL 的表数据。

- partition_name

  您想要从中导出数据的分区。默认情况下，如果未指定此参数，StarRocks 会导出表的所有分区数据。

- column_name

  您想要从中导出数据的列。您使用此参数指定的列顺序可以与表的架构不同。默认情况下，如果未指定此参数，StarRocks 会导出表的所有列数据。

- export_path

  您想要将表数据导出到的位置。如果该位置包含路径，请确保路径以斜杠（/）结尾。否则，路径中最后一个斜杠（/）后的部分将被用作导出文件名的前缀。如果未指定文件名前缀，默认使用 data_ 作为文件名前缀。

- opt_properties

  您可以为导出任务配置的可选属性。

  语法：

  ```SQL
  [PROPERTIES ("<key>"="<value>", ...)]
  ```

  |属性|描述|
|---|---|
  |column_separator|要在导出的文件中使用的列分隔符。默认值：\t。|
  |line_delimiter|要在导出的文件中使用的行分隔符。默认值：\n.|
  |load_mem_limit|每个单独 BE 上的导出任务允许的最大内存。单位：字节。默认最大内存为 2 GB。|
  |timeout|导出任务超时之前的时间量。单位：第二。默认值：86400，表示1天。|
  |include_query_id|指定导出文件的名称是否包含query_id。有效值：true 和 false。 true 表示文件名中包含query_id，false 表示文件名中不包含query_id。|

- 搭配 BROKER 使用

  在 v2.4 及之前的版本中，输入 `WITH BROKER "<broker_name>"` 来指定您想要使用的代理（Broker）。从 v2.5 版本开始，不再需要指定代理，但仍需保留 `WITH BROKER` 关键词。有关详细信息，请参见[使用 EXPORT \"> 背景信息](../../../unloading/Export.md#background-information)导出数据。

- broker_properties

  用于验证源数据的信息。根据数据源的不同，认证信息也会有所不同。更多信息，请参阅 [BROKER LOAD](../../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 文档。

## 示例

### 将表的所有数据导出到 HDFS

以下示例将 testTbl 表的所有数据导出到 HDFS 集群的 hdfs://<hdfs_host>:<hdfs_port>/a/b/c/ 路径：

```SQL
EXPORT TABLE testTbl 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/" 
WITH BROKER
(
        "username"="xxx",
        "password"="yyy"
);
```

### 导出表的特定分区数据到 HDFS

以下示例将 testTbl 表的两个分区，p1 和 p2，的数据导出到 HDFS 集群的 hdfs://<hdfs_host>:<hdfs_port>/a/b/c/ 路径：

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

### 将表的所有数据导出到 HDFS 并指定列分隔符

以下示例将 testTbl 表的所有数据导出到 HDFS 集群的 hdfs://<hdfs_host>:<hdfs_port>/a/b/c/ 路径，并指定使用逗号（,）作为列分隔符：

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

以下示例将 testTbl 表的所有数据导出到 HDFS 集群的 hdfs://<hdfs_host>:<hdfs_port>/a/b/c/ 路径，并指定使用 \x01（Hive 默认支持的列分隔符）作为列分隔符：

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

以下示例将 testTbl 表的所有数据导出到 HDFS 集群的 hdfs://<hdfs_host>:<hdfs_port>/a/b/c/ 路径，并指定使用 testTbl_ 作为导出文件名的前缀：

```SQL
EXPORT TABLE testTbl 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/testTbl_" 
WITH BROKER;
```

### 导出数据到 AWS S3

以下示例将 testTbl 表的所有数据导出到 AWS S3 存储桶的 s3-package/export/ 路径：

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
