---
displayed_sidebar: "Chinese"
---

# 创建表

## 描述

在StarRocks中创建新表。

> **注意**
>
> 此操作需要在目标数据库上具有CREATE TABLE特权。

## 语法

```plaintext
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] [database.]table_name
(column_definition1[, column_definition2, ...]
[, index_definition1[, index_definition12,]])
[ENGINE = [olap|mysql|elasticsearch|hive|hudi|iceberg|jdbc]]
[key_desc]
[COMMENT "table comment"]
[partition_desc]
[distribution_desc]
[rollup_index]
[ORDER BY (column_definition1,...)]
[PROPERTIES ("key"="value", ...)]
[BROKER PROPERTIES ("key"="value", ...)]
```

## 参数

### column_definition

语法：

```SQL
col_name col_type [agg_type] [NULL | NOT NULL] [DEFAULT "default_value"] [AUTO_INCREMENT] [AS generation_expr]
```

**col_name**：列名称。

请注意，通常情况下，您不能创建以下初始为`__op`或`__row`的列名，因为这些名称格式在StarRocks中保留用于特殊用途，创建这些列可能导致未定义的行为。如果确实需要创建这样的列，请将FE动态参数[`allow_system_reserved_names`](../../../administration/Configuration.md#allow_system_reserved_names)设置为`TRUE`。

**col_type**：列类型。具体的列信息，如类型和范围：

- TINYINT（1字节）：范围从-2^7 + 1到2^7 - 1。
- SMALLINT（2字节）：范围从-2^15 + 1到2^15 - 1。
- INT（4字节）：范围从-2^31 + 1到2^31 - 1。
- BIGINT（8字节）：范围从-2^63 + 1到2^63 - 1。
- LARGEINT（16字节）：范围从-2^127 + 1到2^127 - 1。
- FLOAT（4字节）：支持科学计数法。
- DOUBLE（8字节）：支持科学计数法。
- DECIMAL[(precision, scale)]（16字节）

  - 默认值：DECIMAL(10, 0)
  - precision：1〜38
  - scale：0〜precision
  - 整数部分：precision - scale

    不支持科学计数法。

- DATE（3字节）：范围从0000-01-01到9999-12-31。
- DATETIME（8字节）：范围从0000-01-01 00:00:00到9999-12-31 23:59:59。
- CHAR[(长度)]：固定长度字符串。范围：1〜255。默认值：1。
- VARCHAR[(长度)]：可变长度字符串。默认值为1。单位：字节。在StarRocks 2.1之前的版本中，`长度`的值范围是1-65533。[预览] 在StarRocks 2.1及更高版本中，`长度`的值范围是1-1048576。
- HLL（1~16385字节）：对于HLL类型，无需指定长度或默认值。长度将根据数据聚合在系统内控制。HLL列只能由[hll_union_agg](../../sql-functions/aggregate-functions/hll_union_agg.md)、[Hll_cardinality](../../sql-functions/scalar-functions/hll_cardinality.md)和[hll_hash](../../sql-functions/aggregate-functions/hll_hash.md)查询或使用。
- BITMAP：位图类型不需要指定长度或默认值。它代表一组无符号的bigint数字。最大元素可以高达2^64 - 1。

**agg_type**：聚合类型。如果未指定，则此列为键列。如果指定，则为值列。支持的聚合类型如下：


- SUM、MAX、MIN、REPLACE
- HLL_UNION（仅适用于HLL类型）
- BITMAP_UNION（仅适用于BITMAP）
- REPLACE_IF_NOT_NULL：这意味着仅当导入数据为非空值时，才会替换数据。如果为null值，StarRocks将保留原始值。

> 注意
>
> - 当导入聚合类型为BITMAP_UNION的列时，其原始数据类型必须为TINYINT、SMALLINT、INT和BIGINT。
> - 如果在创建表时，REPLACE_IF_NOT_NULL列指定为NOT NULL，则StarRocks仍会将数据转换为NULL，而不会向用户报告错误。通过这种方式，用户可以导入所选列。

此聚合类型仅适用于键描述类型为AGGREGATE KEY的聚合表。

**NULL | NOT NULL**：列是否允许为`NULL`。默认情况下，在使用重复键、聚合或唯一键表的表中，对所有列均使用`NULL`。在使用主键表的表中，默认情况下，值列使用`NULL`，而键列使用`NOT NULL`。如果原始数据中包含`NULL`值，请使用`\N`表示。StarRocks在数据加载期间将`\N`视为`NULL`。

**DEFAULT "default_value"**：列的默认值。当将数据加载到StarRocks时，如果映射到列的源字段为空，则StarRocks会自动在列中填充默认值。您可以通过以下方式指定默认值：

- **DEFAULT current_timestamp**：使用当前时间作为默认值。有关更多信息，请参阅[current_timestamp()](../../sql-functions/date-time-functions/current_timestamp.md)。
- **DEFAULT `<default_value>`**：使用列数据类型的给定值作为默认值。例如，如果列的数据类型为VARCHAR，则可以指定VARCHAR字符串，例如beijing，作为默认值，如`DEFAULT "beijing"`所示。请注意，默认值不能是以下类型之一：ARRAY、BITMAP、JSON、HLL和BOOLEAN。
- **DEFAULT (\<expr\>)**：使用给定函数返回的结果作为默认值。仅支持[uuid()](../../sql-functions/utility-functions/uuid.md)和[uuid_numeric()](../../sql-functions/utility-functions/uuid_numeric.md)表达式。

**AUTO_INCREMENT**：指定`AUTO_INCREMENT`列。`AUTO_INCREMENT`列的数据类型必须为BIGINT。自增ID从1开始，步长为1。有关`AUTO_INCREMENT`列的更多信息，请参阅[AUTO_INCREMENT](../../sql-statements/auto_increment.md)。从v3.0版开始，StarRocks支持`AUTO_INCREMENT`列。

**AS generation_expr**：指定生成的列及其表达式。[生成列](../generated_columns.md)可用于预先计算和存储表达式的结果，从而显着加速具有相同复杂表达式的查询。从v3.1版开始，StarRocks支持生成列。

### index_definition

在创建表时，只能创建位图索引。有关参数描述和使用注意事项，详见[位图索引](../../../using_starrocks/Bitmap_index.md#create-a-bitmap-index)。

```SQL
INDEX index_name (col_name[, col_name, ...]) [USING BITMAP] COMMENT 'xxxxxx'
```

### ENGINE 类型

默认值：`olap`。如果未指定此参数，将默认创建OLAP表（StarRocks原生表）。

可选值：`mysql`、`elasticsearch`、`hive`、`jdbc`（2.3及更高版本）、`iceberg`和`hudi`（2.2及更高版本）。如果要创建用于查询外部数据源的外部表，请指定`CREATE EXTERNAL TABLE`，并将`ENGINE`设置为这些值中的任何一个。有关更多信息，请参见[外部表](../../../data_source/External_table.md)。

**从v3.0版开始，我们建议您使用目录来查询来自Hive、Iceberg、Hudi和JDBC数据源的数据。外部表已弃用。有关详细信息，请参见[Hive目录](../../../data_source/catalog/hive_catalog.md)、[Iceberg目录](../../../data_source/catalog/iceberg_catalog.md)、[Hudi目录](../../../data_source/catalog/hudi_catalog.md)和[JDBC目录](../../../data_source/catalog/jdbc_catalog.md)。**

**从v3.1版开始，StarRocks支持在Iceberg目录中创建Parquet格式的表，并且您可以使用[INSERT INTO](../data-manipulation/INSERT.md)向这些Parquet格式的Iceberg表插入数据。请参阅[创建Iceberg表](../../../data_source/catalog/iceberg_catalog.md#create-an-iceberg-table)。**

**从v3.2版开始，StarRocks支持在Hive目录中创建Parquet格式的表，并且您可以使用[INSERT INTO](../data-manipulation/INSERT.md)向这些Parquet格式的Hive表插入数据。请参阅[创建Hive表](../../../data_source/catalog/hive_catalog.md#create-a-hive-table)。**

- 对于MySQL，请指定以下属性：

    ```plaintext
    PROPERTIES (
        "host" = "mysql_server_host",
        "port" = "mysql_server_port",
```plaintext
"user" = "your_user_name",
"password" = "your_password",
"database" = "database_name",
"table" = "table_name"
```

注意:

在MySQL中，"table_name"应指示真实表名。相比之下，在CREATE TABLE语句中的"table_name"表示 StarRocks 上此 MySQL 表的名称。它们可以是不同的，也可以是相同的。

在StarRocks中创建 MySQL 表的目的是访问 MySQL 数据库。StarRocks 本身不维护或存储任何 MySQL 数据。

- 对于 Elasticsearch，请指定以下属性：

```plaintext
PROPERTIES (

"hosts" = "http://192.168.0.1:8200,http://192.168.0.2:8200",
"user" = "root",
"password" = "root",
"index" = "tindex",
"type" = "doc"
)
```

  - `hosts`：用于连接您的 Elasticsearch 集群的 URL。您可以指定一个或多个 URL。
  - `user`：用于登录启用基本身份验证的 Elasticsearch 集群的 root 用户帐户。
  - `password`：前述 root 帐户的密码。
  - `index`：Elasticsearch 集群中 StarRocks 表的索引。索引名称与 StarRocks 表名称相同。您可以将此参数设置为 StarRocks 表的别名。
  - `type`：索引类型。默认值为`doc`。

- 对于 Hive，请指定以下属性：

```plaintext
PROPERTIES (

    "database" = "hive_db_name",
    "table" = "hive_table_name",
    "hive.metastore.uris" = "thrift://127.0.0.1:9083"
)
```

这里，database 是 Hive 表中对应数据库的名称。Table 是 Hive 表的名称。hive.metastore.uris 是服务器地址。

- 对于 JDBC，请指定以下属性：

```plaintext
PROPERTIES (
"resource"="jdbc0",
"table"="dest_tbl"
)
```

`resource` 是 JDBC 资源名称，`table` 是目标表。

- 对于 Iceberg，请指定以下属性：

   ```plaintext
    PROPERTIES (
    "resource" = "iceberg0", 
    "database" = "iceberg", 
    "table" = "iceberg_table"
    )
    ```

    `resource` 是 Iceberg 资源名称。`database` 是 Iceberg 数据库。`table` 是 Iceberg 表。

- 对于 Hudi，请指定以下属性：

```plaintext
    PROPERTIES (
    "resource" = "hudi0", 
    "database" = "hudi", 
    "table" = "hudi_table" 
    )
```

### key_desc

语法：

```SQL
key_type(k1[,k2 ...])
```

数据按照指定键列进行排序，不同键类型具有不同的属性：

- AGGREGATE KEY：按照指定聚合类型，键列中相同内容将聚合到值列中。通常适用于财务报表和多维分析等业务场景。
- UNIQUE KEY/PRIMARY KEY：按照导入顺序，键列中相同内容将在值列中替换。可用于对键列进行添加、删除、修改和查询。
- DUPLICATE KEY：存在于键列中的相同内容也同时存在于 StarRocks 中。可用于存储详细数据或无聚合属性的数据。**DUPLICATE KEY 是默认类型。数据将根据键列进行排序。**

> **注意**
>
> 除 AGGREGATE KEY 外的其他 `key_type` 创建表时，值列无需指定聚合类型。

### COMMENT

您可以在创建表时添加表注释，此为可选项。请注意，COMMENT 必须放置在 `key_desc` 之后，否则无法创建表。

从 v3.1 开始，您可以使用 `ALTER TABLE <table_name> COMMENT = "new table comment"` 来修改表注释。

### partition_desc

分区描述可使用以下方式：

#### 动态创建分区

[动态分区](../../../table_design/dynamic_partitioning.md) 提供了对分区的生存期（TTL）管理。StarRocks 自动提前创建新分区，删除已过期的分区，以确保数据的新鲜度。要启用此功能，可以在表创建时配置与动态分区相关的属性。

#### 逐个创建分区

**仅指定分区的上界**

语法：

```sql
PARTITION BY RANGE ( <partitioning_column1> [, <partitioning_column2>, ... ] )
  PARTITION <partition1_name> VALUES LESS THAN ("<upper_bound_for_partitioning_column1>" [ , "<upper_bound_for_partitioning_column2>", ... ] )
  [ ,
  PARTITION <partition2_name> VALUES LESS THAN ("<upper_bound_for_partitioning_column1>" [ , "<upper_bound_for_partitioning_column2>", ... ] )
  , ... ] 
)
```

注意：
- 请在分区性的指定键列和指定值范围上使用分区。

- 分区名称仅支持 [A-z0-9_]

- Range 分区中的列仅支持以下类型：`TINYINT`, `SMALLINT`, `INT`, `BIGINT`, `LARGEINT`, `DATE`, 以及 `DATETIME`.

- 分区是左闭右开的。第一个分区的左边界是最小值。

- NULL 值仅存储在包含最小值的分区中。当删除包含最小值的分区后，将不再能导入 NULL 值。

- 分区列可以是单个列或多个列。分区值为默认最小值。

- 当仅指定一个列作为分区列时，您可以将 `MAXVALUE` 设置为最近分区的分区列的上界。

  ```SQL
  PARTITION BY RANGE (pay_dt) (
    PARTITION p1 VALUES LESS THAN ("20210102"),
    PARTITION p2 VALUES LESS THAN ("20210103"),
    PARTITION p3 VALUES LESS THAN MAXVALUE
  )
  ```

请注意：

- 分区通常用于管理与时间相关的数据。
- 当需要数据回溯时，您可能需要考虑在必要时清空第一个分区以后再添加分区。

**同时指定分区的下限和上限**

语法：

```SQL
PARTITION BY RANGE ( <partitioning_column1> [, <partitioning_column2>, ... ] )
(
    PARTITION <partition_name1> VALUES [( "<lower_bound_for_partitioning_column1>" [ , "<lower_bound_for_partitioning_column2>", ... ] ), ( "<upper_bound_for_partitioning_column1?" [ , "<upper_bound_for_partitioning_column2>", ... ] ) ) 
    [,
    PARTITION <partition_name2> VALUES [( "<lower_bound_for_partitioning_column1>" [ , "<lower_bound_for_partitioning_column2>", ... ] ), ( "<upper_bound_for_partitioning_column1>" [ , "<upper_bound_for_partitioning_column2>", ... ] ) ) 
    , ...]
)
```

注意：
- 固定 Range 比 LESS THAN 更灵活。您可以自定义左右分区。
- 固定 Range 在其他方面与 LESS THAN 相同。
- 当仅指定一个列作为分区列时，您可以将 `MAXVALUE` 设置为最近分区的分区列的上界。

  ```SQL
  PARTITION BY RANGE (pay_dt) (
    PARTITION p202101 VALUES [("20210101"), ("20210201")),
    PARTITION p202102 VALUES [("20210201"), ("20210301")),
    PARTITION p202103 VALUES [("20210301"), (MAXVALUE))
  )
  ```

#### 批量创建多个分区

语法

- 如果分区列为日期类型。

    ```sql
    PARTITION BY RANGE (<partitioning_column>) (
        START ("<start_date>") END ("<end_date>") EVERY (INTERVAL <N> <time_unit>)
    )
    ```

- 如果分区列为整数类型。

    ```sql
    PARTITION BY RANGE (<partitioning_column>) (
        START ("<start_integer>") END ("<end_integer>") EVERY (<partitioning_granularity>)
    )
    ```

描述

您可以在 `START()` 和 `END()` 中指定起始值和结束值，并在 `EVERY()` 中指定时间单位或分区粒度，以批量创建多个分区。

- 分区列可以是日期或整数类型。
- 如果分区列为日期类型，您需要使用 `INTERVAL` 关键字来指定时间间隔。您可以将时间单位指定为小时（自 v3.0 开始）、天、周、月或年。分区名称的命名规则与动态分区相同。

有关更多信息，请参见[数据分布](../../../table_design/Data_distribution.md)。

### distribution_desc

StarRocks 支持哈希分桶和随机分桶。如果您不配置分桶，StarRocks 将使用随机分桶，并默认自动设置桶的数量。

- 随机分桶（自 v3.1 开始）
```
对于分区中的数据，StarRocks会将数据随机分布到所有存储桶中，这不是基于特定列值的。如果您希望StarRocks自动确定存储桶的数量，您不需要指定任何存储桶配置。如果选择手动指定存储桶数量，语法如下：

```SQL
DISTRIBUTED BY RANDOM BUCKETS <num>
```

然而需要注意的是，当您查询大量数据并频繁使用某些列作为条件列时，随机分桶提供的查询性能可能并不理想。在这种情况下，建议使用哈希分桶。因为只需要扫描和计算少量存储桶，可以显著提高查询性能。

**注意事项**
- 您只能使用随机分桶来创建重复键表。
- 您不能为随机存储分桶的表指定[共位连接组](../../../using_starrocks/Colocate_join.md)。
- [Spark加载](../../../loading/SparkLoad.md)不能用于将数据加载到随机分桶的表中。
- 自StarRocks v2.5.7起，创建表时不需要设置存储桶数量。StarRocks会自动设置存储桶数量。如果您想设置此参数，请参阅 [确定存储桶数量](../../../table_design/Data_distribution.md#determine-the-number-of-buckets)。

有关更多信息，请参阅[随机分桶](../../../table_design/Data_distribution.md#random-bucketing-since-v31)。

- 哈希分桶

  语法：

  ```SQL
  DISTRIBUTED BY HASH (k1[,k2 ...]) [BUCKETS num]
  ```

  根据分区数据的哈希值和存储桶的数量，可以将分区中的数据细分为存储桶。建议您选择符合以下两个要求的列作为存储桶列。

  - 高基数列，例如ID
  - 经常在查询中用作过滤器的列

  如果不存在这样的列，则可以根据查询的复杂性来确定存储桶列。

  - 如果查询复杂，建议您选择高基数列作为存储桶列，以确保数据在存储桶之间平衡分布，并提高集群资源利用率。
  - 如果查询相对简单，建议您选择经常用作查询条件的列作为存储桶列，以提高查询效率。

  如果使用一个存储桶列无法使分区数据均匀分布到每个存储桶中，可以选择多个存储桶列（最多三个）。有关更多信息，请参阅[选择存储桶列](../../../table_design/Data_distribution.md#hash-bucketing)。

  **注意事项**：

  - **创建表时必须指定其存储桶列**。
  - 无法更新存储桶列的值。
  - 一旦指定了存储桶列，就无法修改。
  - 自StarRocks v2.5.7起，创建表时不需要设置存储桶数量。StarRocks会自动设置存储桶数量。如果您想设置此参数，请参阅 [确定存储桶数量](../../../table_design/Data_distribution.md#determine-the-number-of-buckets)。

### ORDER BY

自版本3.0起，主键和排序键在主键表中是分离的。排序键由`ORDER BY`关键字指定，并且可以是任何列的排列和组合。

> **注意**
>
> 如果指定了排序键，将根据排序键构建前缀索引；如果未指定排序键，将根据主键构建前缀索引。

### PROPERTIES

#### 指定初始存储介质、自动存储冷却时间、副本数

如果引擎类型是`OLAP`，您可以在创建表时指定初始存储介质(`storage_medium`)、自动存储冷却时间(`storage_cooldown_time`)或时间间隔(`storage_cooldown_ttl`)，以及副本数(`replication_num`)。

属性生效范围：如果表只有一个分区，则属性属于表。如果表分为多个分区，则属性属于每个分区。当需要为指定的分区配置不同的属性时，可以在表创建后执行[ALTER TABLE ... ADD PARTITION 或 ALTER TABLE ... MODIFY PARTITION](../data-definition/ALTER_TABLE.md)。

**设置初始存储介质和自动存储冷却时间**

```sql
PROPERTIES (
    "storage_medium" = "[SSD|HDD]",
    { "storage_cooldown_ttl" = "<num> { YEAR | MONTH | DAY | HOUR } "
    | "storage_cooldown_time" = "yyyy-MM-dd HH:mm:ss" }
)
```

- `storage_medium`：初始存储介质，可以设置为`SSD`或`HDD`。请确保您明确指定的存储介质类型与BE静态参数`storage_root_path`中为StarRocks群集指定的BE磁盘类型一致。<br />

    如果FE配置项`enable_strict_storage_medium_check`设置为`true`，系统在创建表时严格检查BE磁盘类型。如果您在CREATE TABLE中指定的存储介质与BE磁盘类型不一致，则会返回错误信息“Failed to find enough host in all backends with storage medium is SSD|HDD.”，导致表创建失败。如果`enable_strict_storage_medium_check`设置为`false`，系统将忽略此错误并强制创建表。但是，数据加载后集群磁盘空间可能分布不均匀。<br />

    从v2.3.6、v2.4.2、v2.5.1以及v3.0开始，系统会根据BE磁盘类型自动推断存储介质，如果未明确指定`storage_medium`。<br />

  - 在以下情况下，系统会自动将此参数设置为SSD：

    - BE报告的磁盘类型(`storage_root_path`)仅包含SSD。
    - BE报告的磁盘类型(`storage_root_path`)同时包含SSD和HDD。请注意，从v2.3.10、v2.4.5、v2.5.4以及v3.0开始，当BE报告的`storage_root_path`同时包含SSD和HDD，并且指定了`storage_cooldown_time`属性，系统会将`storage_medium`设置为SSD。

  - 在以下情况下，系统会自动将此参数设置为HDD：

    - BE报告的磁盘类型(`storage_root_path`)仅包含HDD。
    - 从v2.3.10、v2.4.5、v2.5.4以及v3.0开始， 当BE报告的`storage_root_path`同时包含SSD和HDD，并且未指定`storage_cooldown_time`属性，系统会将`storage_medium`设置为HDD。

- `storage_cooldown_ttl`或`storage_cooldown_time`：自动存储冷却时间或时间间隔。自动存储冷却指的是自动将数据从SSD迁移至HDD。此功能仅在初始存储介质为SSD时有效。

  **参数**

  - `storage_cooldown_ttl`：此表中分区的自动存储冷却时间间隔。如果您需要在一定时间间隔后将最近的分区保留在SSD上并自动将旧分区冷却到HDD上，您可以使用此参数。每个分区的自动存储冷却时间是使用此参数的值加上分区上限时间的结果。

  支持的值有`<num> YEAR`、`<num> MONTH`、`<num> DAY`和`<num> HOUR`。`<num>`是非负整数。默认值为空，表示不会自动执行存储冷却。

  例如，创建表时指定值为`"storage_cooldown_ttl"="1 DAY"`，并存在范围为`[2023-08-01 00:00:00,2023-08-02 00:00:00)`的分区`p20230801`。此分区的自动存储冷却时间为`2023-08-03 00:00:00`，即`2023-08-02 00:00:00 + 1 DAY`。如果创建表时指定值为`"storage_cooldown_ttl"="0 DAY"`，则此分区的自动存储冷却时间为`2023-08-02 00:00:00`。

  - `storage_cooldown_time`：表从SSD冷却到HDD的自动存储冷却时间（**绝对时间**）。指定的时间必须晚于当前时间。格式："yyyy-MM-dd HH:mm:ss"。当需要为指定的分区配置不同的属性时，可以执行[ALTER TABLE ... ADD PARTITION 或 ALTER TABLE ... MODIFY PARTITION](../data-definition/ALTER_TABLE.md)。

**用法**

- 自动存储冷却相关参数的比较如下：
  - `storage_cooldown_ttl`：指定表中分区的自动存储冷却时间间隔的表属性。系统会在分区的`此参数值加上分区上限时间`时自动触发存储冷却。因此自动存储冷却是以分区为粒度进行的，更加灵活。
- `storage_cooldown_time`: 表属性，用于指定此表的自动存储冷却时间（**绝对时间**）。此外，您还可以在创建表后为指定分区配置不同的属性。
  - `storage_cooldown_second`: 静态 FE（前端）参数，用于指定集群内所有表的自动存储冷却延迟。

- 表属性 `storage_cooldown_ttl` 或 `storage_cooldown_time` 优先于 FE 静态参数 `storage_cooldown_second`.
- 在配置这些参数时，需要指定 `"storage_medium = "SSD"`。
- 如果您不配置这些参数，则不会自动执行自动存储冷却。
- 执行 `SHOW PARTITIONS FROM <table_name>` 以查看每个分区的自动存储冷却时间。

**限制**

- 不支持表达式和列表分区。
- 分区列需要为日期类型。
- 不支持多个分区列。
- 不支持主键表。

**设置分区中每个数据片的副本数**

`replication_num`：分区中每个表的副本数。默认数：`3`。

```sql
PROPERTIES (
    "replication_num" = "<num>"
)
```

#### 为列添加布隆过滤器索引

如果引擎类型为 OLAP，则可以指定一个列来采用布隆过滤器索引。

在使用布隆过滤器索引时，适用以下限制：

- 您可以为重复键或主键表的所有列创建布隆过滤器索引。对于聚合表或唯一键表，您只能为键列创建布隆过滤器索引。
- TINYINT、FLOAT、DOUBLE 和 DECIMAL 列不支持创建布隆过滤器索引。
- 布隆过滤器索引只能提升包含 `in` 和 `=` 等运算符的查询性能，例如包含 `Select xxx from table where x in {}` 和 `Select xxx from table where column = xxx` 的查询。此列中的离散值越多，查询就越准确。

有关更多信息，请参见 [布隆过滤器索引](../../../using_starrocks/Bloomfilter_index.md)。

```SQL
PROPERTIES (
    "bloom_filter_columns"="k1,k2,k3"
)
```

#### 使用 Colocate Join

如果要使用 Colocate Join 属性，请在 `properties` 中指定它。

```SQL
PROPERTIES (
    "colocate_with"="table1"
)
```

#### 配置动态分区

如果要使用动态分区属性，请在属性中指定它。

```SQL
PROPERTIES (

    "dynamic_partition.enable" = "true|false",
    "dynamic_partition.time_unit" = "DAY|WEEK|MONTH",
    "dynamic_partition.start" = "${integer_value}",
    "dynamic_partition.end" = "${integer_value}",
    "dynamic_partition.prefix" = "${string_value}",
    "dynamic_partition.buckets" = "${integer_value}"
```

**`PROPERTIES`**

| 参数                        | 必需     | 描述                                                       |
| -------------------------- | -------- | ---------------------------------------------------------- |
| dynamic_partition.enable    | 否       | 是否启用动态分区。有效值：`TRUE` 和 `FALSE`。默认值：`TRUE`。 |
| dynamic_partition.time_unit | 是       | 动态创建分区的时间粒度。这是一个必需参数。有效值：`DAY`、`WEEK` 和 `MONTH`。时间粒度确定动态创建分区的后缀格式。<br/>  - 如果值为 `DAY`，则动态创建分区的后缀格式为 `yyyyMMdd`。例如，分区名后缀为 `20200321`。<br/>  - 如果值为 `WEEK`，则动态创建分区的后缀格式为 `yyyy_ww`，例如 `2020_13` 代表 2020 年的第 13 周。<br/>  - 如果值为 `MONTH`，则动态创建分区的后缀格式为 `yyyyMM`，例如 `202003`。 |
| dynamic_partition.start     | 否       | 动态分区的起始偏移量。该参数的值必须是负整数。这个偏移量之前的分区将根据 `dynamic_partition.time_unit`（由其确定当前天、周或月）进行删除。默认值是 `Integer.MIN_VALUE`，即 -2147483648，这意味着历史分区不会被删除。 |
| dynamic_partition.end       | 是       | 动态分区的结束偏移量。该参数的值必须是正整数。从当前天、周或月到结束偏移量之间的分区将提前创建。 |
| dynamic_partition.prefix    | 否       | 动态分区名称的前缀。默认值：`p`。 |
| dynamic_partition.buckets   | 否       | 每个动态分区的桶数。默认值与由保留字 `BUCKETS` 确定的桶数或 StarRocks 自动设置的桶数相同。 |

#### 为配置了随机分桶的表指定桶大小

自 v3.2 起，对于配置了随机分桶的表，您可以通过在表创建时使用 `PROPERTIES` 中的 `bucket_size` 参数来指定桶的大小。默认大小为 `1024 * 1024 * 1024 B`（1 GB），最大大小为 4 GB。

```sql
PROPERTIES (
    "bucket_size" = "3221225472"
)
```

#### 设置数据压缩算法

您可以在创建表时通过添加属性 `compression` 来指定表的数据压缩算法。

`compression` 的有效值为：

- `LZ4`：LZ4 算法。
- `ZSTD`：Zstandard 算法。
- `ZLIB`：zlib 算法。
- `SNAPPY`：Snappy 算法。

有关如何选择合适的数据压缩算法的更多信息，请参见 [数据压缩](../../../table_design/data_compression.md)。

#### 设置数据加载的写入仲裁

如果您的 StarRocks 集群具有多个数据副本，则可以为表设置不同的写入仲裁，即在 StarRocks 判断加载任务成功之前需要多少个副本返回加载成功。您可以在创建表时通过添加属性 `write_quorum` 来指定写入仲裁。此属性从版本 v2.5 开始支持。

`write_quorum` 的有效值为：

- `MAJORITY`：默认值。当大多数数据副本返回加载成功时，StarRocks 返回加载任务成功。否则，StarRocks 返回加载任务失败。
- `ONE`：当一个数据副本返回加载成功时，StarRocks 返回加载任务成功。否则，StarRocks 返回加载任务失败。
- `ALL`：当所有数据副本返回加载成功时，StarRocks 返回加载任务成功。否则，StarRocks 返回加载任务失败。

> **注意**
>
> - 为加载设置较低的写入仲裁会增加数据不可访问甚至丢失的风险。例如，在一个包含两个副本的 StarRocks 集群中，您将数据加载到一个写入仲裁为一个的表中，并且数据仅成功加载到一个副本中。尽管 StarRocks 判断了加载任务成功，但数据只有一个副本存活。如果存储加载数据数据片的服务器宕机，则这些数据片中的数据将无法访问。如果服务器的磁盘损坏，则数据将丢失。
> - StarRocks 仅在所有数据副本都返回状态后才返回加载任务状态。当存在加载状态未知的副本时，StarRocks 不会返回加载任务状态。在一个副本中，加载超时也被视为加载失败。

#### 在副本间指定数据写入和复制模式

如果您的 StarRocks 集群具有多个数据副本，则可以在 `PROPERTIES` 中指定 `replicated_storage` 参数来配置副本间的数据写入和复制模式。

- `true`（从 v3.0 开始的默认值）表示“单领导复制”，这意味着数据仅写入主副本。其他副本从主副本同步数据。这种模式显著降低了由于数据写入多个副本而导致的 CPU 成本。从 v2.5 开始支持。
- `false`（v2.5 的默认值）表示“无领导复制”，这意味着数据直接写入多个副本，不区分主副本和次要副本。CPU 成本是副本数的倍数。

在大多数情况下，使用默认值会获得更好的数据写入性能。如果要更改副本间的数据写入和复制模式，请运行 ALTER TABLE 命令。示例：

```sql
    ALTER TABLE example_db.my_table
    SET ("replicated_storage" = "false");
```

#### 批量创建 Rollup

您可以在创建表时批量创建 Rollup。

语法：

```SQL
ROLLUP (rollup_name (column_name1, column_name2, ...)
[FROM from_index_name]
[PROPERTIES ("key"="value", ...)],...)
```

#### 为 View Delta Join 查询重写定义唯一键约束和外键约束

要在 View Delta Join 场景中启用查询重写，必须为要在 Delta Join 中加入的表定义唯一键约束 `unique_constraints` 和外键约束 `foreign_key_constraints`。有关详细信息，请参见 [异步物化视图 - 在 View Delta Join 场景中重写查询](../../../using_starrocks/query_rewrite_with_materialized_views.md#query-delta-join-rewrite)。

```SQL
PROPERTIES (
    "unique_constraints" = "<unique_key>[, ...]",
    "foreign_key_constraints" = "

    (<child_column>[, ...]) 
    REFERENCES 
    [catalog_name].[database_name].<parent_table_name>(<parent_column>[, ...])
    [;...]
    "


- `child_column`: 表的外键。您可以定义多个`child_column`。
- `catalog_name`: 表联接所在的目录名称。如果未指定此参数，则使用默认目录。
- `database_name`: 表联接所在的数据库名称。如果未指定此参数，则使用当前数据库。
- `parent_table_name`: 要联接的表名称。
- `parent_column`: 要联接的列。它们必须是相应表的主键或唯一键。

> **注意**
>
> - `unique_constraints` 和 `foreign_key_constraints` 仅用于查询重写。在将数据加载到表中时，并不保证外键约束检查。您必须确保加载到表中的数据满足约束条件。
> - 主键表的主键或唯一键表的唯一键默认为对应的 `unique_constraints`。您无需手动设置。
> - 表的`foreign_key_constraints`中的`child_column`必须引用另一张表的`unique_constraints`中的`unique_key`。
> - `child_column` 和 `parent_column` 的数量必须一致。
> - `child_column` 和相应的`parent_column`的数据类型必须匹配。

#### 为 StarRocks 共享数据集群创建云原生表

要[使用您的 StarRocks 共享数据集群](../../../deployment/shared_data/s3.md#use-your-shared-data-starrocks-cluster)，您必须使用以下属性创建云原生表：

```SQL
PROPERTIES (
    "datacache.enable" = "{ true | false }",
    "datacache.partition_duration" = "<string_value>",
    "enable_async_write_back" = "{ true | false }"
)
```

- `datacache.enable`: 是否启用本地磁盘缓存。默认值：`true`。

  - 当将此属性设置为`true`时，即将要加载的数据同时写入对象存储和本地磁盘（作为查询加速的缓存）。
  - 当将此属性设置为`false`时，即将数据仅加载到对象存储中。

  > **注意**
  >
  > 若要启用本地磁盘缓存，您必须在 BE 配置项`storage_root_path`中指定磁盘的目录。详情请参见[BE Configuration items](../../../administration/Configuration.md#be-configuration-items)。

- `datacache.partition_duration`: 热数据的有效持续时间。启用本地磁盘缓存后，所有数据都加载到缓存中。当缓存已满时，StarRocks 会从缓存中删除最近不常用的数据。当查询需要扫描已删除的数据时，StarRocks 会检查数据是否在有效期内。如果数据在有效期内，StarRocks 会再次将其加载到缓存中。如果数据不在有效期内，StarRocks 不会将其加载到缓存中。此属性是一个字符串值，可以用以下单位指定： `YEAR`、`MONTH`、`DAY` 和 `HOUR`，例如 `7 DAY` 和 `12 HOUR`。如果未指定，则所有数据都作为热数据缓存。

  > **注意**
  >
  > 仅当`datacache.enable`设置为`true`时，此属性可用。

- `enable_async_write_back`: 是否允许数据异步写入对象存储。默认值：`false`。

  - 当将此属性设置为`true`时，加载任务在数据写入本地磁盘缓存后立即返回成功，并将数据异步写入对象存储。这可以提高加载性能，但也会在系统出现潜在故障时影响数据可靠性。
  - 当将此属性设置为`false`时，加载任务仅在数据写入对象存储和本地磁盘缓存后才返回成功。这保证了更高的可用性，但会降低加载性能。

#### 设置快速模式演进

`fast_schema_evolution`: 是否启用表的快速模式演进。有效值为`TRUE`（默认值）或`FALSE`。启用快速模式演进可以增加模式更改的速度，并在添加或删除列时减少资源使用率。当前，此属性仅在表创建时启用，并且在表创建后无法通过[ALTER TABLE](../../sql-statements/data-definition/ALTER_TABLE.md)进行修改。此参数自 v3.2.0 版本起受支持。
  > **注意**
  >
  > - StarRocks 共享数据集群不支持此参数。
  > - 如果需要在集群级别配置快速模式演进（例如，在 StarRocks 集群中停用快速模式演进），您可以设置 FE 动态参数[`fast_schema_evolution`](../../../administration/Configuration.md#fast_schema_evolution)。

## 示例

### 创建一个使用哈希分桶和列式存储的聚合表

```SQL
CREATE TABLE example_db.table_hash
(
    k1 TINYINT,
    k2 DECIMAL(10, 2) DEFAULT "10.5",
    v1 CHAR(10) REPLACE,
    v2 INT SUM
)
ENGINE=olap
AGGREGATE KEY(k1, k2)
COMMENT "my first starrocks table"
DISTRIBUTED BY HASH(k1)
PROPERTIES ("storage_type"="column");
```

### 创建一个聚合表并设置存储介质和冷却时间

```SQL
CREATE TABLE example_db.table_hash
(
    k1 BIGINT,
    k2 LARGEINT,
    v1 VARCHAR(2048) REPLACE,
    v2 SMALLINT SUM DEFAULT "10"
)
ENGINE=olap
UNIQUE KEY(k1, k2)
DISTRIBUTED BY HASH (k1, k2)
PROPERTIES(
    "storage_type"="column",
    "storage_medium" = "SSD",
    "storage_cooldown_time" = "2015-06-04 00:00:00"
);
```

或

```SQL
CREATE TABLE example_db.table_hash
(
    k1 BIGINT,
    k2 LARGEINT,
    v1 VARCHAR(2048) REPLACE,
    v2 SMALLINT SUM DEFAULT "10"
)
ENGINE=olap
PRIMARY KEY(k1, k2)
DISTRIBUTED BY HASH (k1, k2)
PROPERTIES(
    "storage_type"="column",
    "storage_medium" = "SSD",
    "storage_cooldown_time" = "2015-06-04 00:00:00"
);
```

### 创建一个使用范围分区、哈希分桶和基于列的存储的重复键表，并设置存储介质和冷却时间

小于

```SQL
CREATE TABLE example_db.table_range
(
    k1 DATE,
    k2 INT,
    k3 SMALLINT,
    v1 VARCHAR(2048),
    v2 DATETIME DEFAULT "2014-02-04 15:36:00"
)
ENGINE=olap
DUPLICATE KEY(k1, k2, k3)
PARTITION BY RANGE (k1)
(
    PARTITION p1 VALUES LESS THAN ("2014-01-01"),
    PARTITION p2 VALUES LESS THAN ("2014-06-01"),
    PARTITION p3 VALUES LESS THAN ("2014-12-01")
)
DISTRIBUTED BY HASH(k2)
PROPERTIES(
    "storage_medium" = "SSD", 
    "storage_cooldown_time" = "2015-06-04 00:00:00"
);
```

注：

此语句将创建三个数据分区：

```SQL
( {    MIN     },   {"2014-01-01"} )
[ {"2014-01-01"},   {"2014-06-01"} )
[ {"2014-06-01"},   {"2014-12-01"} )
```

这些范围之外的数据将不会被加载。

固定范围

```SQL
CREATE TABLE table_range
(
    k1 DATE,
    k2 INT,
    k3 SMALLINT,
    v1 VARCHAR(2048),
    v2 DATETIME DEFAULT "2014-02-04 15:36:00"
)
ENGINE=olap
DUPLICATE KEY(k1, k2, k3)
PARTITION BY RANGE (k1, k2, k3)
(
    PARTITION p1 VALUES [("2014-01-01", "10", "200"), ("2014-01-01", "20", "300")),
    PARTITION p2 VALUES [("2014-06-01", "100", "200"), ("2014-07-01", "100", "300"))
)
DISTRIBUTED BY HASH(k2)
PROPERTIES(
    "storage_medium" = "SSD"
);
```

### 创建一个MySQL外部表

```SQL
CREATE EXTERNAL TABLE example_db.table_mysql
(
    k1 DATE,
    k2 INT,
    k3 SMALLINT,
    k4 VARCHAR(2048),
    k5 DATETIME
)
ENGINE=mysql
PROPERTIES
(
    "host" = "127.0.0.1",
    "port" = "8239",
    "user" = "mysql_user",
    "password" = "mysql_passwd",
    "database" = "mysql_db_test",
    "table" = "mysql_table_test"
)
```

```SQL
创建一个包含HLL列的表

```SQL
创建表example_db.example_table
(
    k1 TINYINT,
    k2 DECIMAL(10, 2) 默认 "10.5",
    v1 HLL HLL_UNION,
    v2 HLL HLL_UNION
)
ENGINE=olap
根据HASH(k1)分布
属性("storage_type"="column");
```

创建一个包含BITMAP_UNION聚合类型的表

原始数据类型的`v1`和`v2`列必须是TINYINT、SMALLINT或INT。

```SQL
CREATE TABLE example_db.example_table
(
    k1 TINYINT,
    k2 DECIMAL(10, 2) DEFAULT "10.5",
    v1 BITMAP BITMAP_UNION,
    v2 BITMAP BITMAP_UNION
)
ENGINE=olap
聚合键(k1, k2)
根据HASH(k1)分布
属性("storage_type"="column");
```

创建支持Colocate Join的两个表

```SQL
CREATE TABLE `t1` 
(
    `id` int(11) 注释 "",
    `value` varchar(8) 注释 ""
) 
ENGINE=OLAP
重复键(`id`)
根据HASH(`id`)分布
属性 
(
    "colocate_with" = "t1"
);

CREATE TABLE `t2` 
(
    `id` int(11) 注释 "",
    `value` varchar(8) 注释 ""
) 
ENGINE=OLAP
重复键(`id`)
根据HASH(`id`)分布
属性 
(
    "colocate_with" = "t1"
);
```

创建带有位图索引的表

```SQL
CREATE TABLE example_db.table_hash
(
    k1 TINYINT,
    k2 DECIMAL(10, 2) 默认 "10.5",
    v1 CHAR(10) 替换,
    v2 INT SUM,
    索引 k1_idx (k1) 使用位图 注释 'xxxxxx'
)
ENGINE=olap
聚合键(k1, k2)
注释"我的第一个starrocks表"
根据HASH(k1)分布
属性("storage_type"="column");
```

创建动态分区表

在FE配置中必须启用动态分区函数("dynamic_partition.enable" = "true")。更多信息，请参见[配置动态分区](#configure-dynamic-partitions)。

此示例对接下来的三天创建分区，并删除三天前创建的分区。例如，如果今天是2020-01-08，那么将创建以下名称的分区：p20200108、p20200109、p20200110、p20200111，它们的范围是：

```明文
[类型：[日期]; 键：[2020-01-08]; ‥类型：[日期]; 键：[2020-01-09]; )
[类型：[日期]; 键：[2020-01-09]; ‥类型：[日期]; 键：[2020-01-10]; )
[类型：[日期]; 键：[2020-01-10]; ‥类型：[日期]; 键：[2020-01-11]; )
[类型：[日期]; 键：[2020-01-11]; ‥类型：[日期]; 键：[2020-01-12]; )
```

```SQL
CREATE TABLE example_db.dynamic_partition
(
    k1 DATE,
    k2 INT,
    k3 SMALLINT,
    v1 VARCHAR(2048),
    v2 DATETIME 默认 "2014-02-04 15:36:00"
)
ENGINE=olap
重复键(k1, k2, k3)
按RANGE(k1)分区
(
    分区p1 VALUES LESS THAN ("2014-01-01"),
    分区p2 VALUES LESS THAN ("2014-06-01"),
    分区p3 VALUES LESS THAN ("2014-12-01")
)
根据HASH(k2)分布
属性(
    "storage_medium" = "SSD",
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "DAY",
    "dynamic_partition.start" = "-3",
    "dynamic_partition.end" = "3",
    "dynamic_partition.prefix" = "p",
    "dynamic_partition.buckets" = "10"
);
```

创建一个批量创建多个分区并将整数类型列指定为分区列的表

在下面的示例中，分区列`datekey`的类型是INT。所有分区通过一个简单的分区子句`START ("1") END ("5") EVERY (1)`来创建。所有分区的范围从`1`开始，到`5`结束，分区粒度为`1`:
> **注意**
> 
> **START()**和**END()**中的分区列值需要用引号括起来，而**EVERY()**中的分区粒度则不需要用引号括起来。

```SQL
CREATE TABLE site_access (
    datekey INT,
    site_id INT,
    city_code SMALLINT,
    user_name VARCHAR(32),
    pv BIGINT 默认 '0'
)
ENGINE=olap
重复键(datekey, site_id, city_code, user_name)
按RANGE(datekey)分区 (START ("1") END ("5") EVERY (1)
)
根据HASH(site_id)分布
属性("replication_num" = "3");
```

创建Hive外部表

在创建Hive外部表之前，您必须已创建了Hive资源和数据库。更多信息，请参见[外部表](../../../data_source/External_table.md#deprecated-hive-external-table)。

```SQL
CREATE EXTERNAL TABLE example_db.table_hive
(
    k1 TINYINT,
    k2 VARCHAR(50),
    v INT
)
ENGINE=hive
属性
(
    "resource" = "hive0",
    "database" = "hive_db_name",
    "table" = "hive_table_name"
);
```

创建具有主键并指定排序键的表

假设您需要从用户地址和最后活动时间等维度实时分析用户行为。创建表时，可以将`user_id`列定义为主键，将`address`和`last_active`列的组合定义为排序键。

```SQL
create table users (
    user_id bigint 不为空,
    name string 不为空,
    email string 空,
    address string 空,
    age tinyint 空,
    sex tinyint 空,
    last_active datetime,
    property0 tinyint 不为空,
    property1 tinyint 不为空,
    property2 tinyint 不为空,
    property3 tinyint 不为空
) 
主键 (`user_id`)
根据HASH(`user_id`)分布
排序依据(`address`,`last_active`)
属性(
    "replication_num" = "3",
    "enable_persistent_index" = "true"
);
```

## 参考

- [SHOW CREATE TABLE](../data-manipulation/SHOW_CREATE_TABLE.md)
- [SHOW TABLES](../data-manipulation/SHOW_TABLES.md)
- [USE](USE.md)
- [ALTER TABLE](ALTER_TABLE.md)
- [DROP TABLE](DROP_TABLE.md)