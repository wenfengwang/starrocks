---
displayed_sidebar: "Chinese"
---

# 创建表

This article describes how to create tables in StarRocks and perform related operations.

> **Note**
>
> Only users with the [CREATE DATABASE](../administration/privilege_item.md) permission under the default_catalog can create databases. Only users with the CREATE TABLE permission for the database can create tables in that database.

## Connect to StarRocks

After successfully [deploying the StarRocks cluster](../quick_start/deploy_with_docker.md), you can connect to the `query_port` of any FE node (default is `9030`) through the MySQL client to connect to StarRocks. StarRocks has a built-in `root` user with a default empty password.

```shell
mysql -h <fe_host> -P9030 -u root
```

## Create a Database

Create the `example_db` database.

> **Note**
>
> When specifying variable names such as database names, table names, and column names, if reserved keywords are used, they must be enclosed in backticks (`) to avoid potential errors. For the list of reserved keywords in StarRocks, please refer to [Keywords](../sql-reference/sql-statements/keywords.md#reserved-keywords).

```sql
CREATE DATABASE example_db;
```

You can use the `SHOW DATABASES;` command to view all databases in the current StarRocks cluster.

```Plain Text
MySQL [(none)]> SHOW DATABASES;

+--------------------+
| Database           |
+--------------------+
| _statistics_       |
| example_db         |
| information_schema |
+--------------------+
3 rows in set (0.00 sec)
```

> Note: Similar to the table structure in MySQL, `information_schema` contains metadata information of the current StarRocks cluster, but some statistical information is not complete. We recommend using commands like `DESC table_name` to obtain database metadata information.

## Create a Table

Create a table in the newly created database.

StarRocks supports [multiple data models](../table_design/table_types/table_types.md) to suit different application scenarios. The following example is based on the [detail table model](../table_design/table_types/duplicate_key_table.md) to write the table creation statement.

For more table creation syntax, refer to [CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md).

```sql
use example_db;
CREATE TABLE IF NOT EXISTS `detailDemo` (
    `recruit_date`  DATE           NOT NULL COMMENT "YYYY-MM-DD",
    `region_num`    TINYINT        COMMENT "range [-128, 127]",
    `num_plate`     SMALLINT       COMMENT "range [-32768, 32767]",
    `tel`           INT            COMMENT "range [-2147483648, 2147483647]",
    `id`            BIGINT         COMMENT "range [-2^63 + 1 ~ 2^63 - 1]",
    `password`      LARGEINT       COMMENT "range [-2^127 + 1 ~ 2^127 - 1]",
    `name`          CHAR(20)       NOT NULL COMMENT "range char(m),m in (1-255)",
    `profile`       VARCHAR(500)   NOT NULL COMMENT "upper limit value 1048576 bytes",
    `hobby`         STRING         NOT NULL COMMENT "upper limit value 65533 bytes",
    `leave_time`    DATETIME       COMMENT "YYYY-MM-DD HH:MM:SS",
    `channel`       FLOAT          COMMENT "4 bytes",
    `income`        DOUBLE         COMMENT "8 bytes",
    `account`       DECIMAL(12,4)  COMMENT "",
    `ispass`        BOOLEAN        COMMENT "true/false"
) ENGINE=OLAP
DUPLICATE KEY(`recruit_date`, `region_num`)
PARTITION BY RANGE(`recruit_date`)
(
    PARTITION p20220311 VALUES [('2022-03-11'), ('2022-03-12')),
    PARTITION p20220312 VALUES [('2022-03-12'), ('2022-03-13')),
    PARTITION p20220313 VALUES [('2022-03-13'), ('2022-03-14')),
    PARTITION p20220314 VALUES [('2022-03-14'), ('2022-03-15')),
    PARTITION p20220315 VALUES [('2022-03-15'), ('2022-03-16'))
)
DISTRIBUTED BY HASH(`recruit_date`, `region_num`);
```

> Note
>
> * In StarRocks, field names are case insensitive, but table names are case sensitive.
> * Starting from version 3.1, you can omit the bucketing key (DISTRIBUTED BY clause) when creating a table. StarRocks defaults to using random distribution, distributing data randomly across all buckets in the partition. For more information, refer to [Random Distribution](../table_design/Data_distribution.md#random-distribution-since-v31).

### Explanation of Table Creation Statement

#### Sorting Key

When organizing and storing data within StarRocks tables, specified columns are used as the sorting key. In the detail model, the `DUPLICATE KEY` specifies the sorting columns. In the example above, the `recruit_date` and `region_num` are the sorting columns.

> Note: Sorting columns should be defined before other columns when creating a table. For detailed descriptions of sorting keys and how to set tables for different data models, refer to [Sorting Key](../table_design/Sort_key.md).

#### Field Types

StarRocks tables support various field types. Besides the field types mentioned in the example, it also supports [BITMAP type](../using_starrocks/Using_bitmap.md), [HLL type](../using_starrocks/Using_HLL.md), [ARRAY type](../sql-reference/sql-statements/data-types/Array.md). For detailed field types, refer to the [Data Types section](../sql-reference/sql-statements/data-types/BIGINT.md).

> Note: When creating a table, you should use the most appropriate types. For instance, integer data should not be stored using string types; INT type is sufficient and should not be replaced with BIGINT type. Precise data types can better utilize the database's performance.

#### Partitioning and Bucketing

The `PARTITION` keyword is used to [create partitions](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md#partition_desc) for tables. In the example above, range partitioning is used based on `recruit_date`, with a partition created for each day from the 11th to the 15th. StarRocks supports dynamic partition generation, as detailed in [Dynamic Partition Management](../table_design/dynamic_partitioning.md). **To optimize query performance in production environments, we strongly recommend devising a reasonable data partitioning plan for tables.**

The `DISTRIBUTED` keyword is used to [create bucketing](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md#distribution_desc). In the example above, `recruit_date` and `region_num` are used as bucketing columns, and starting from version 2.5.7, StarRocks supports automatic bucket quantity setting, eliminating the need for manual bucket amount setting. For details, refer to [Determining Bucket Quantity](../table_design/Data_distribution.md#determining-bucket-quantity).

Reasonable partitioning and bucketing designs during table creation can optimize query performance. For information on how to select partitioning and bucketing columns, refer to [Data Distribution](../table_design/Data_distribution.md).

#### Data Models

The `DUPLICATE` keyword indicates that the current table is a detail model, with the `KEY` list specifying the sorting columns for the table. StarRocks supports multiple data models, including the [detail model](../table_design/table_types/duplicate_key_table.md), [aggregate model](../table_design/table_types/aggregate_table.md), [update model](../table_design/table_types/unique_key_table.md), and [primary key model](../table_design/table_types/primary_key_table.md). Different models are suitable for various business scenarios, and selecting the right model can optimize query efficiency.

#### Index

StarRocks默认会给Key列创建稀疏索引加速查询，具体规则见[排序键](../table_design/Sort_key.md)。支持的索引类型有[Bitmap 索引](../using_starrocks/Bitmap_index.md)，[Bloomfilter索引](../using_starrocks/Bloomfilter_index.md)等。

> 注意：索引创建对表模型和列有要求，详细说明见对应索引介绍章节。

#### ENGINE类型

默认ENGINE类型为 `olap`，对应StarRocks集群内部表。其他可选项包括`mysql`，`elasticsearch`，`hive`，`jdbc`（2.3及以后），`hudi`（2.2及以后）以及`iceberg`，分别代表所创建的表为相应类型的[外部表](../data_source/External_table.md)。

## 查看表信息

您可以通过SQL命令查看表的相关信息。

* 查看当前数据库中所有的表

```sql
SHOW TABLES;
```

* 查看表的结构

```sql
DESC table_name;
```

示例：

```sql
DESC detailDemo;
```

* 查看建表语句

```sql
SHOW CREATE TABLE table_name;
```

示例：

```sql
SHOW CREATE TABLE detailDemo;
```

<br/>

## 修改表结构

StarRocks支持多种DDL操作。

您可以通过[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md#schema-change)命令可以修改表的Schema，包括增加列，删除列，修改列类型（暂不支持修改列名称），改变列顺序。

### 增加列

例如，在以上创建的表中，在 `ispass` 列后新增一列 `uv`，类型为 BIGINT，默认值为 `0`。

```sql
ALTER TABLE detailDemo ADD COLUMN uv BIGINT DEFAULT '0' after ispass;
```

### 删除列

删除以上步骤新增的列。

> 注意
>
> 如果您通过上述步骤添加了`uv`，请务必删除此列以保证后续Quick Start内容可以执行。

```sql
ALTER TABLE detailDemo DROP COLUMN uv;
```

### 查看修改表结构作业状态

修改表结构为异步操作。提交成功后，您可以通过以下命令查看作业状态。

```sql
SHOW ALTER TABLE COLUMN\G;
```

当作业状态为FINISHED，则表示作业完成，新的表结构修改已生效。

修改Schema完成之后，您可以通过以下命令查看最新的表结构。

```sql
DESC table_name;
```

示例如下：

```Plain Text
MySQL [example_db]> desc detailDemo;

+--------------+-----------------+------+-------+---------+-------+
| Field        | Type            | Null | Key   | Default | Extra |
+--------------+-----------------+------+-------+---------+-------+
| recruit_date | DATE            | No   | true  | NULL    |       |
| region_num   | TINYINT         | Yes  | true  | NULL    |       |
| num_plate    | SMALLINT        | Yes  | false | NULL    |       |
| tel          | INT             | Yes  | false | NULL    |       |
| id           | BIGINT          | Yes  | false | NULL    |       |
| password     | LARGEINT        | Yes  | false | NULL    |       |
| name         | CHAR(20)        | No   | false | NULL    |       |
| profile      | VARCHAR(500)    | No   | false | NULL    |       |
| hobby        | VARCHAR(65533)  | No   | false | NULL    |       |
| leave_time   | DATETIME        | Yes  | false | NULL    |       |
| channel      | FLOAT           | Yes  | false | NULL    |       |
| income       | DOUBLE          | Yes  | false | NULL    |       |
| account      | DECIMAL64(12,4) | Yes  | false | NULL    |       |
| ispass       | BOOLEAN         | Yes  | false | NULL    |       |
| uv           | BIGINT          | Yes  | false | 0       |       |
+--------------+-----------------+------+-------+---------+-------+
15 rows in set (0.00 sec)
```

### 取消修改表结构

您可以通过以下命令取消当前正在执行的作业。

```sql
CANCEL ALTER TABLE COLUMN FROM table_name\G;
```

## 创建用户并授权

`example_db`数据库创建完成之后，您可以创建`test`用户，并授予其`example_db`的读写权限。

```sql
CREATE USER 'test' IDENTIFIED by '123456';
GRANT ALL on example_db.* to test;
```

通过登录被授权的`test`账户，就可以操作`example_db`数据库。

```bash
mysql -h 127.0.0.1 -P9030 -utest -p123456
```

<br/>

## 下一步

表创建成功后，您可以[导入并查询数据](../quick_start/Import_and_query.md)。