---
displayed_sidebar: English
---

# 使用 INSERT INTO FILES 导出数据

本主题将介绍如何通过使用 INSERT INTO FILES 将 StarRocks 中的数据导出至远程存储。

从 v3.2 版本开始，StarRocks 支持使用表函数 FILES() 来在远程存储中定义一个可写的文件。您可以将 FILES() 与 INSERT 语句相结合，从而将 StarRocks 中的数据导出到您的远程存储中。

与 StarRocks 支持的其它数据导出方式相比，通过 INSERT INTO FILES 导出数据提供了一个更加统一且易用的接口。您可以使用与加载数据时相同的语法，直接将数据导出到远程存储。此外，这种方法支持通过提取指定列的值，将数据文件存储到不同的存储路径，从而以分区的方式管理导出的数据。

> **注意**
- 请注意，通过 INSERT INTO FILES 导出数据不支持到本地文件系统的数据导出。
- 目前，INSERT INTO FILES 仅支持导出为 Parquet 文件格式的数据。

## 准备工作

以下示例创建了一个名为 unload 的数据库和一个名为 sales_records 的表，作为后续教程中将会用到的数据对象。您也可以使用自己的数据替代。

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

sales_records 表包含了每笔交易的交易 ID（record_id）、销售人员（seller）、商店 ID（store_id）、销售时间（sales_time）和销售金额（sales_amt），并且根据销售时间（sales_time）进行了每日分区。

您还需要准备一个具有写权限的远程存储系统。以下示例使用了启用了简单认证方式的 HDFS 集群。关于支持的远程存储系统和认证方式的更多信息，请参见 [SQL 参考文档 - FILES()](../sql-reference/sql-functions/table-functions/files.md)。

## 数据导出

INSERT INTO FILES 支持将数据导出到单个文件或多个文件中。您可以通过为这些数据文件指定不同的存储路径来进一步分区。

在使用 INSERT INTO FILES 导出数据时，您必须手动设置压缩算法，这可以通过 `compression` 属性来完成。关于 StarRocks 支持的数据压缩算法的更多信息，请参阅[数据压缩](../table_design/data_compression.md)文档。

### 导出数据到多个文件

默认情况下，INSERT INTO FILES 将数据导出为多个数据文件，每个文件大小约为 1 GB。您可以通过 max_file_size 属性来配置文件的大小。

以下示例将 sales_records 表中的所有数据行导出为多个以 data1 为前缀的 Parquet 文件，每个文件的大小设置为 1 KB。

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

### 导出数据到不同路径下的多个文件

您还可以通过 partition_by 属性，根据指定列的值，将数据文件导出到不同的存储路径下进行分区。

以下示例将`sales_records`表中的所有数据行导出到HDFS集群的**/unload/partitioned/**路径下的多个Parquet文件中。这些文件按照`sales_time`列的值存储在不同的子路径下。

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

### 导出数据到单个文件

要将数据导出到一个单独的数据文件中，您需要将 single 属性设置为 true。

以下示例将 sales_records 表中的所有数据行导出为一个以 data2 为前缀的单个 Parquet 文件。

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

## 另请参阅

- 关于 INSERT 使用方法的更多说明，请参阅[SQL reference - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。
- 关于 FILES() 使用方法的更多说明，请参阅 [SQL reference - FILES()](../sql-reference/sql-functions/table-functions/files.md)。
