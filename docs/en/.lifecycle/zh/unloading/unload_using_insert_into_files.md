---
displayed_sidebar: English
---

# 使用 INSERT INTO FILES 卸载数据

本主题介绍如何使用 INSERT INTO FILES 将数据从 StarRocks 卸载到远程存储。

从 v3.2 版本开始，StarRocks 支持使用表函数 FILES() 定义远程存储中的可写文件。然后，您可以将 FILES() 与 INSERT 语句结合使用，将数据从 StarRocks 卸载到您的远程存储。

与 StarRocks 支持的其他数据导出方法相比，使用 INSERT INTO FILES 卸载数据提供了一个更加统一、易于使用的接口。您可以使用与加载数据时相同的语法直接将数据卸载到远程存储中。此外，该方法支持通过提取指定列的值，将数据文件存储在不同的存储路径中，从而允许您以分区布局的方式管理导出的数据。

> **注意**
- 请注意，使用 INSERT INTO FILES 卸载数据不支持导出数据到本地文件系统。
- 目前，INSERT INTO FILES 仅支持卸载 Parquet 文件格式的数据。

## 准备工作

以下示例创建了一个名为 `unload` 的数据库和一个名为 `sales_records` 的表，作为后续教程中可以使用的数据对象。您也可以使用自己的数据。

```SQL
CREATE DATABASE unload;
USE unload;
CREATE TABLE sales_records(
    record_id     BIGINT,
    seller        STRING,
    store_id      INT,
    sales_time    DATETIME,
    sales_amt     DOUBLE
)
DUPLICATE KEY(record_id)
PARTITION BY date_trunc('day', sales_time)
DISTRIBUTED BY HASH(record_id);

INSERT INTO sales_records
VALUES
    (220313001,"Amy",1,"2022-03-13 12:00:00",8573.25),
    (220314002,"Bob",2,"2022-03-14 12:00:00",6948.99),
    (220314003,"Amy",1,"2022-03-14 12:00:00",4319.01),
    (220315004,"Carl",3,"2022-03-15 12:00:00",8734.26),
    (220316005,"Carl",3,"2022-03-16 12:00:00",4212.69),
    (220317006,"Bob",2,"2022-03-17 12:00:00",9515.88);
```

表 `sales_records` 包含每笔交易的交易 ID `record_id`、销售员 `seller`、商店 ID `store_id`、时间 `sales_time` 和销售额 `sales_amt`。它根据 `sales_time` 按天进行分区。

您还需要准备一个具有写入权限的远程存储系统。以下示例使用启用了简单认证方法的 HDFS 集群。有关支持的远程存储系统和认证方法的更多信息，请参阅 [SQL 参考 - FILES()](../sql-reference/sql-functions/table-functions/files.md)。

## 卸载数据

INSERT INTO FILES 支持将数据卸载到单个文件或多个文件中。您可以通过为这些数据文件指定不同的存储路径来进一步分区。

使用 INSERT INTO FILES 卸载数据时，您必须手动设置压缩算法，这可以通过 `compression` 属性完成。有关 StarRocks 支持的数据压缩算法的更多信息，请参阅 [数据压缩](../table_design/data_compression.md)。

### 将数据卸载到多个文件

默认情况下，INSERT INTO FILES 将数据卸载到多个数据文件中，每个文件的大小为 1 GB。您可以使用 `max_file_size` 属性来配置文件大小。

以下示例将 `sales_records` 中的所有数据行卸载为以 `data1` 为前缀的多个 Parquet 文件。每个文件的大小为 1 KB。

```SQL
INSERT INTO 
FILES(
    "path" = "hdfs://xxx.xx.xxx.xx:9000/unload/data1",
    "format" = "parquet",
    "hadoop.security.authentication" = "simple",
    "username" = "xxxxx",
    "password" = "xxxxx",
    "compression" = "lz4",
    "max_file_size" = "1KB"
)
SELECT * FROM sales_records;
```

### 将数据卸载到不同路径下的多个文件

您还可以通过使用 `partition_by` 属性提取指定列的值来对不同存储路径中的数据文件进行分区。

以下示例将 `sales_records` 中的所有数据行卸载为存储在 HDFS 集群的 **/unload/partitioned/** 路径下的多个 Parquet 文件。这些文件存储在通过 `sales_time` 列的值区分的不同子路径中。

```SQL
INSERT INTO 
FILES(
    "path" = "hdfs://xxx.xx.xxx.xx:9000/unload/partitioned/",
    "format" = "parquet",
    "hadoop.security.authentication" = "simple",
    "username" = "xxxxx",
    "password" = "xxxxx",
    "compression" = "lz4",
    "partition_by" = "sales_time"
)
SELECT * FROM sales_records;
```

### 将数据卸载到单个文件

要将数据卸载到一个单独的数据文件中，您必须将 `single` 属性设置为 `true`。

以下示例将 `sales_records` 中的所有数据行卸载为一个前缀为 `data2` 的单个 Parquet 文件。

```SQL
INSERT INTO 
FILES(
    "path" = "hdfs://xxx.xx.xxx.xx:9000/unload/data2",
    "format" = "parquet",
    "hadoop.security.authentication" = "simple",
    "username" = "xxxxx",
    "password" = "xxxxx",
    "compression" = "lz4",
    "single" = "true"
)
SELECT * FROM sales_records;
```

## 参见

- 有关 INSERT 使用的更多说明，请参阅 [SQL 参考 - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。
- 有关 FILES() 使用的更多说明，请参阅 [SQL 参考 - FILES()](../sql-reference/sql-functions/table-functions/files.md)。