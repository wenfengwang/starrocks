---
displayed_sidebar: "Chinese"
---

# 使用 INSERT INTO FILES 卸载数据

本主题描述了如何使用 INSERT INTO FILES 从 StarRocks 卸载数据到远程存储。

从 v3.2 开始，StarRocks 支持使用 TABLE 函数 FILES() 来定义远程存储中的可写文件。然后，您可以将 FILES() 与 INSERT 语句结合使用，将数据从 StarRocks 卸载到您的远程存储中。

与 StarRocks 支持的其他数据导出方法相比，使用 INSERT INTO FILES 卸载数据提供了一个更统一、易于使用的接口。您可以直接使用与加载数据相同的语法将数据卸载到您的远程存储中。此外，此方法支持通过提取指定列的值将数据文件存储在不同的存储路径中，允许您以分区布局管理导出的数据。

> **注意**
>
> - 请注意，使用 INSERT INTO FILES 卸载数据不支持将数据导出到本地文件系统。
> - 目前， INSERT INTO FILES 仅支持以 Parquet 文件格式卸载数据。

## 准备工作

以下示例创建了一个名为 `unload` 的数据库和一个名为 `sales_records` 的表，作为下面的教程中可以使用的数据对象。您也可以使用自己的数据。

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

表 `sales_records` 包含了每笔交易的交易ID `record_id`、销售人员 `seller`、店铺ID `store_id`、时间 `sales_time` 和销售金额 `sales_amt`。根据 `sales_time` 按天进行分区。

您还需要准备一个具有写入权限的远程存储系统。以下示例使用启用了简单身份验证方法的 HDFS 集群。有关支持的远程存储系统和凭据方法的更多信息，请参阅 [SQL 参考 - FILES()](../sql-reference/sql-functions/table-functions/files.md)。

## 卸载数据

INSERT INTO FILES 支持将数据卸载到单个文件或多个文件中。您可以通过为这些数据文件指定单独的存储路径来进一步分区这些数据文件。

使用 INSERT INTO FILES 卸载数据时，您必须使用属性 `compression` 手动设置压缩算法。有关 StarRocks 支持的数据压缩算法的更多信息，请参阅 [数据压缩](../table_design/data_compression.md)。

### 卸载数据到多个文件中

默认情况下，INSERT INTO FILES 将数据卸载到多个大小为 1 GB 的数据文件中。您可以使用属性 `max_file_size` 配置文件大小。

以下示例将 `sales_records` 中的所有数据行作为以 `data1` 为前缀的多个 Parquet 文件卸载。每个文件的大小为 1 KB。

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

### 卸载数据到不同路径下的多个文件

您还可以使用属性 `partition_by` 通过提取指定列的值将数据文件分区到不同的存储路径中。

以下示例将 `sales_records` 中的所有数据行作为 HDFS 集群中 **/unload/partitioned/** 路径下的多个 Parquet 文件卸载。这些文件存储在不同的子路径中，由列 `sales_time` 的值加以区分。

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

### 卸载数据到单个文件中

要将数据卸载到单个数据文件中，您必须将属性 `single` 指定为 `true`。

以下示例将 `sales_records` 中的所有数据行作为一个以 `data2` 为前缀的单个 Parquet 文件卸载。

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

- 有关 INSERT 的更多使用说明，请参阅 [SQL 参考 - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。
- 有关 FILES() 的更多使用说明，请参阅 [SQL 参考 - FILES()](../sql-reference/sql-functions/table-functions/files.md)。