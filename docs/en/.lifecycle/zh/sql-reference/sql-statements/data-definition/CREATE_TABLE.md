---
displayed_sidebar: English
---

# 创建表

## 描述

在 StarRocks 中创建一个新表。

> **注意**
>
> 此操作需要在目标数据库上具有 CREATE TABLE 权限。

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

**col_name**：列名。

需要注意的是，通常情况下，您不能创建以 `__op` 或 `__row` 开头的列，因为这些名称格式在 StarRocks 中保留用于特殊目的，创建这样的列可能导致未定义的行为。如果确实需要创建这样的列，请将 FE 动态参数 [`allow_system_reserved_names`](../../../administration/FE_configuration.md#allow_system_reserved_names) 设置为 `TRUE`。

**col_type**：列类型。具体的列信息，比如类型和范围：

- TINYINT（1 字节）：范围从 -2^7 + 1 到 2^7 - 1。
- SMALLINT（2 字节）：范围从 -2^15 + 1 到 2^15 - 1。
- INT（4 字节）：范围从 -2^31 + 1 到 2^31 - 1。
- BIGINT（8 字节）：范围从 -2^63 + 1 到 2^63 - 1。
- LARGEINT（16 字节）：范围从 -2^127 + 1 到 2^127 - 1。
- FLOAT（4 字节）：支持科学计数法。
- DOUBLE（8 字节）：支持科学计数法。
- DECIMAL[(precision, scale)]（16 字节）

  - 默认值：DECIMAL(10, 0)
  - 精度：1 ~ 38
  - 刻度：0 ~ 精度
  - 整数部分：精度 - 刻度

    不支持科学计数法。

- DATE（3 字节）：范围从 0000-01-01 到 9999-12-31。
- DATETIME（8 字节）：范围从 0000-01-01 00:00:00 到 9999-12-31 23:59:59。
- CHAR[(length)]：固定长度字符串。范围：1 ~ 255。默认值：1。
- VARCHAR[(length)]：可变长度字符串。默认值为 1。单位：字节。在 StarRocks 2.1 之前的版本中，`length` 的取值范围为 1–65533。[预览] 在 StarRocks 2.1 及更高版本中，`length` 的取值范围为 1–1048576。
- HLL（1~16385 字节）：对于 HLL 类型，无需指定长度或默认值。长度将根据数据聚合在系统内进行控制。HLL 列只能由 [hll_union_agg](../../sql-functions/aggregate-functions/hll_union_agg.md)、[Hll_cardinality](../../sql-functions/scalar-functions/hll_cardinality.md) 和 [hll_hash](../../sql-functions/aggregate-functions/hll_hash.md) 查询或使用。
- BITMAP：位图类型不需要指定长度或默认值。它表示一组无符号的 bigint 数字。最大元素可达 2^64 - 1。

**agg_type**：聚合类型。如果未指定，则此列为键列。
如果指定，则为值列。支持的聚合类型如下：

- 总和、最大值、最小值、替换
- HLL_UNION（仅适用于 HLL 类型）
- BITMAP_UNION（仅适用于 BITMAP）
- REPLACE_IF_NOT_NULL：这意味着仅当导入的数据为非 null 值时才会被替换。如果为 null，StarRocks 将保留原来的值。

> 注意
>
> - 导入聚合类型为 BITMAP_UNION 的列时，其原始数据类型必须为 TINYINT、SMALLINT、INT 和 BIGINT。
> - 如果创建表时，REPLACE_IF_NOT_NULL 列指定了 NOT NULL，StarRocks 仍会将数据转换为 NULL，而不会向用户发送错误报告。这样，用户就可以导入选定的列。

此聚合类型仅适用于 key_desc 类型为 AGGREGATE KEY 的 Aggregate 表。

**NULL | NOT NULL**：列是否允许为 `NULL`。默认情况下，对于使用重复键、聚合或唯一键表的表，所有列都指定为 `NULL`。对于使用主键表的表，默认情况下，值列指定为 `NULL`，而键列指定为 `NOT NULL`。如果原始数据中包含 `NULL` 值，请使用 `\N` 表示。StarRocks 在加载数据时将 `\N` 视为 `NULL`。

**DEFAULT "default_value"**：列的默认值。当您将数据加载到 StarRocks 中时，如果映射到该列的源字段为空，StarRocks 会自动填充该列中的默认值。您可以通过以下方式之一指定默认值：

- **DEFAULT current_timestamp**：使用当前时间作为默认值。有关更多信息，请参见 [current_timestamp()](../../sql-functions/date-time-functions/current_timestamp.md)。
- **DEFAULT `<default_value>`**：使用列数据类型的给定值作为默认值。例如，如果列的数据类型为 VARCHAR，则可以将 VARCHAR 字符串（如 beijing）指定为默认值，如 `DEFAULT "beijing"` 所示。请注意，默认值不能是以下任何类型：ARRAY、BITMAP、JSON、HLL 和 BOOLEAN。
- **DEFAULT (\<expr\>)**：使用给定函数返回的结果作为默认值。仅支持 [uuid()](../../sql-functions/utility-functions/uuid.md) 和 [uuid_numeric()](../../sql-functions/utility-functions/uuid_numeric.md) 表达式。

**AUTO_INCREMENT**：指定 `AUTO_INCREMENT` 列。`AUTO_INCREMENT` 列的数据类型必须为 BIGINT。自增 ID 从 1 开始，步长为 1。有关 `AUTO_INCREMENT` 列的更多信息，请参见 [AUTO_INCREMENT](../../sql-statements/auto_increment.md)。从 v3.0 开始，StarRocks 支持 `AUTO_INCREMENT` 列。

**AS generation_expr**：指定生成列及其表达式。[生成列](../generated_columns.md) 可用于预计算和存储表达式的结果，从而显著加快了使用相同复杂表达式的查询速度。从 v3.1 开始，StarRocks 支持生成列。

### index_definition

只能在创建表时创建位图索引。有关参数说明和使用说明的详细信息，请参阅[位图索引](../../../using_starrocks/Bitmap_index.md#create-a-bitmap-index)。

```SQL
INDEX index_name (col_name[, col_name, ...]) [USING BITMAP] COMMENT 'xxxxxx'
```

### ENGINE 类型

默认值：`olap`。如果未指定此参数，默认情况下创建 OLAP 表（StarRocks 原生表）。

可选值：`mysql`、 `elasticsearch` 、`hive`、`jdbc`（2.3 及更高版本）、`iceberg` 和 `hudi` （2.2 及更高版本）。如果要创建外部表以查询外部数据源，请指定 `CREATE EXTERNAL TABLE` 并将 `ENGINE` 设置为这些值中的任何一个。有关更多信息，请参见[外部表](../../../data_source/External_table.md)。

**从 v3.0 开始，我们建议使用 catalog 从 Hive、Iceberg、Hudi 和 JDBC 数据源中查询数据。外部表已被弃用。有关更多信息，请参见[Hive catalog](../../../data_source/catalog/hive_catalog.md)、[Iceberg catalog](../../../data_source/catalog/iceberg_catalog.md)、[Hudi catalog](../../../data_source/catalog/hudi_catalog.md)和[JDBC catalog](../../../data_source/catalog/jdbc_catalog.md)。**

**从 v3.1 开始，StarRocks 支持在 Iceberg 目录中创建 Parquet 格式的表，并且您可以通过[INSERT INTO](../data-manipulation/INSERT.md)将数据插入到这些 Parquet 格式的 Iceberg 表中。请参见[创建 Iceberg 表](../../../data_source/catalog/iceberg_catalog.md#create-an-iceberg-table)。**

**从 v3.2 开始，StarRocks 支持在 Hive 目录中创建 Parquet 格式的表，并且您可以通过[INSERT INTO](../data-manipulation/INSERT.md)将数据插入到这些 Parquet 格式的 Hive 表中。请参见[创建 Hive 表](../../../data_source/catalog/hive_catalog.md#create-a-hive-table)。**

- 对于 MySQL，请指定以下属性：

    ```plaintext
    PROPERTIES (
        "host" = "mysql_server_host",
        "port" = "mysql_server_port",
        "user" = "your_user_name",
        "password" = "your_password",
        "database" = "database_name",
        "table" = "table_name"
    )
    ```

    注意：

    MySQL 中的“table_name”应指示实际表名。相比之下，CREATE TABLE 语句中的“table_name”表示 StarRocks 上这个 MySQL 表的名称。它们可以是不同的，也可以是相同的。

    在 StarRocks 中创建 MySQL 表的目的是访问 MySQL 数据库。StarRocks 本身不维护或存储任何 MySQL 数据。

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

  - `hosts`：用于连接 Elasticsearch 集群的 URL。您可以指定一个或多个 URL。
  - `user`：用于登录已启用基本认证的 Elasticsearch 集群的 root 用户账户。

  - `password`：前一个 root 账号的密码。
  - `index`：Elasticsearch 集群中 StarRocks 表的索引。索引名称与 StarRocks 表名相同。您可以将此参数设置为 StarRocks 表的别名。
  - `type`：索引的类型。默认值为 `doc`。

- 对于 Hive，请指定以下属性：

    ```plaintext
    PROPERTIES (

        "database" = "hive_db_name",
        "table" = "hive_table_name",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
    )
    ```

    这里，database 是 Hive 表中对应数据库的名称。Table 是 Hive 表的名称。 `hive.metastore.uris` 是服务器地址。

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

数据根据指定的键列排序，并且不同的键类型具有不同的属性：

- AGGREGATE KEY：键列中相同的内容将根据指定的聚合类型聚合到值列中。通常适用于财务报表和多维分析等业务场景。
- UNIQUE KEY/PRIMARY KEY：键列中相同的内容将根据导入顺序替换值列中的相同内容。可用于对键列进行添加、删除、修改和查询等操作。
- DUPLICATE KEY：键列中的内容相同，并且同时存在于 StarRocks 中。它可用于存储详细数据或没有聚合属性的数据。**DUPLICATE KEY 是默认类型。数据将根据键列排序。**

> **注意**
>
> 除了 AGGREGATE KEY 外，使用其他 key_type 创建表时，值列不需要指定聚合类型。

### COMMENT

您可以在创建表时添加表注释，这是可选的。请注意，COMMENT 必须放在 `key_desc` 之后。否则，无法创建表。

从 v3.1 开始，您可以使用 `ALTER TABLE <table_name> COMMENT = "new table comment"` 修改表注释。

### partition_desc

分区描述可以通过以下方式使用：

#### 动态创建分区

[动态分区](../../../table_design/dynamic_partitioning.md) 为分区提供生存时间（TTL）管理。StarRocks 会自动提前创建新分区，并删除过期分区，以确保数据新鲜度。要启用此功能，您可以在创建表时配置与动态分区相关的属性。

#### 逐个创建分区

**仅指定分区的上限**

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

请使用指定的键列和指定的值范围进行分区。

- 分区名称仅支持 [A-z0-9_]
- Range 分区中的列仅支持以下类型：TINYINT、SMALLINT、INT、BIGINT、LARGEINT、DATE 和 DATETIME。
- 分区左闭右开。第一个分区的左边界为最小值。
- NULL 值仅存储在包含最小值的分区中。删除包含最小值的分区后，无法再导入 NULL 值。
- 分区列可以是单列，也可以是多列。分区值是默认的最小值。
- 当仅指定一列作为分区列时，可以将最近分区的分区列的上限设置为 `MAXVALUE`。

  ```SQL
  PARTITION BY RANGE (pay_dt) (
    PARTITION p1 VALUES LESS THAN ("20210102"),
    PARTITION p2 VALUES LESS THAN ("20210103"),
    PARTITION p3 VALUES LESS THAN MAXVALUE
  )
  ```

请注意：

- 分区通常用于管理与时间相关的数据。
- 当需要数据回溯时，您可能需要考虑清空第一个分区，以便在必要时添加分区。

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

- 固定范围比 LESS THAN 更灵活。您可以自定义左右分区。
- 固定范围在其他方面与 LESS THAN 相同。
- 当仅指定一列作为分区列时，可以将最近分区的分区列的上限设置为 `MAXVALUE`。

  ```SQL
  PARTITION BY RANGE (pay_dt) (
    PARTITION p202101 VALUES [("20210101"), ("20210201")),
    PARTITION p202102 VALUES [("20210201"), ("20210301")),
    PARTITION p202103 VALUES [("20210301"), (MAXVALUE))
  )
  ```

#### 批量创建多个分区

语法

- 如果分区列是日期类型。

    ```sql
    PARTITION BY RANGE (<partitioning_column>) (
        START ("<start_date>") END ("<end_date>") EVERY (INTERVAL <N> <time_unit>)
    )
    ```

- 如果分区列是整数类型。

    ```sql
    PARTITION BY RANGE (<partitioning_column>) (
        START ("<start_integer>") END ("<end_integer>") EVERY (<partitioning_granularity>)
    )
    ```

描述

您可以在 `START()` 和 `END()` 中指定开始和结束值，并在 `EVERY()` 中指定时间单位或分区粒度，以批量创建多个分区。

- 分区列可以是日期或整数类型。
- 如果分区列是日期类型，则需要使用 `INTERVAL` 关键字指定时间间隔。您可以将时间单位指定为小时（自 v3.0 起）、日、周、月或年。分区的命名约定与动态分区的命名约定相同。

有关详细信息，请参阅 [数据分发](../../../table_design/Data_distribution.md)。

### distribution_desc

StarRocks 支持哈希桶和随机桶。如果您未配置桶，StarRocks 会使用随机桶，并默认自动设置桶数。

- 随机桶（自 v3.1 起）

  对于分区中的数据，StarRocks 会将数据随机分布到所有存储桶中，而不是基于特定的列值。如果希望 StarRocks 自动确定 Bucket 数量，则无需指定任何 Bucket 配置。如果选择手动指定存储桶数量，则语法如下：

  ```SQL
  DISTRIBUTED BY RANDOM BUCKETS <num>
  ```
  
  但是，请注意，当您查询大量数据并频繁使用某些列作为条件列时，随机分桶提供的查询性能可能并不理想。在这种情况下，建议使用哈希分桶。因为只需要扫描和计算少量的存储桶，从而显著提高查询性能。

  **预防措施**
  - 您只能使用随机分桶来创建重复键表。
  - 您不能为随机分桶的表指定 [主机托管组](../../../using_starrocks/Colocate_join.md)。
  - [Spark Load](../../../loading/SparkLoad.md) 不能用于将数据加载到随机分桶的表中。
  - 从 StarRocks v2.5.7 开始，创建表时无需设置桶数。StarRocks 会自动设置存储桶数量。如果您需要设置该参数，请参见设置 [存储桶数量](../../../table_design/Data_distribution.md#set-the-number-of-buckets)。

  有关更多信息，请参阅 [随机分桶](../../../table_design/Data_distribution.md#random-bucketing-since-v31)。

- 哈希桶

  语法：

  ```SQL
  DISTRIBUTED BY HASH (k1[,k2 ...]) [BUCKETS num]
  ```

  分区中的数据可以根据分桶列的哈希值和分桶数量进行细分。建议您选择满足以下两个条件的列作为分桶列。

  - 高基数列，例如 ID
  - 在查询中经常用作筛选器的列

  如果不存在这样的列，您可以根据查询的复杂程度确定存储桶列。

  - 如果查询较复杂，建议选择高基数列作为分桶列，以保证分桶间数据均衡，提高集群资源利用率。
  - 如果查询比较简单，建议您选择经常用作查询条件的列作为分桶列，以提高查询效率。

  如果使用一个桶列无法将数据均匀分布到每个桶中，您可以选择多个桶列（最多三个）。有关更多信息，请参阅 [选择存储桶列](../../../table_design/Data_distribution.md#hash-bucketing)。

  **注意事项**：

  - **创建表时，必须指定其分桶列**。
  - 无法更新分桶列的值。
  - 分桶列一旦指定后就无法修改。
  - 从 StarRocks v2.5.7 开始，创建表时无需设置桶数。StarRocks 会自动设置桶的数量。如果您需要设置这个参数，请参见 [设置桶的数量](../../../table_design/Data_distribution.md#set-the-number-of-buckets)。

### 排序

从 3.0 版本开始，主键和排序键在主键表中分离。排序键由 `ORDER BY` 关键字指定，可以是任何列的排列和组合。

> **注意**
>
> 如果指定了排序键，则根据排序键构建前缀索引；如果未指定排序键，则根据主键构建前缀索引。

### 属性

#### 指定初始存储介质、自动存储冷却时间、副本数

如果引擎类型为 `OLAP`，在创建表时可以指定初始存储介质 (`storage_medium`)、自动存储冷却时间 (`storage_cooldown_time`) 或时间间隔 (`storage_cooldown_ttl`)，以及副本数 (`replication_num`)。

属性生效的范围：如果表只有一个分区，则属性属于该表。如果将表划分为多个分区，则属性属于每个分区。当您需要为指定分区配置不同的属性时，可以在创建表后执行 [ALTER TABLE ...添加分区或更改表...修改分区](../data-definition/ALTER_TABLE.md)。

**设置初始存储介质和自动存储冷却时间**

```sql
PROPERTIES (
    "storage_medium" = "[SSD|HDD]",
    { "storage_cooldown_ttl" = "<num> { YEAR | MONTH | DAY | HOUR } "
    | "storage_cooldown_time" = "yyyy-MM-dd HH:mm:ss" }
)
```

- `storage_medium`：初始存储介质，可以设置为 `SSD` 或 `HDD`。请确保您明确指定的存储介质类型与 BE 静态参数 `storage_root_path` 中指定的 StarRocks 集群的 BE 磁盘类型一致。<br />

    如果 FE 配置项 `enable_strict_storage_medium_check` 设置为 `true`，在创建表时，系统会严格检查 BE 磁盘类型。如果在 CREATE TABLE 中指定的存储介质与 BE 磁盘类型不一致，则会显示错误“Failed to find enough host in all backends with storage medium is SSD|HDD”，并且表创建失败。如果 `enable_strict_storage_medium_check` 设置为 `false`，系统将忽略此错误并强制创建表。但是，加载数据后，群集磁盘空间可能会分布不均匀。<br />

    从 v2.3.6、v2.4.2、v2.5.1 和 v3.0 开始，如果未明确指定，系统会根据 BE 磁盘类型自动推断存储介质 `storage_medium`。<br />

  - 在以下场景中，系统会自动将该参数设置为 SSD：

    - BE 上报的磁盘类型 `storage_root_path` 仅包含 SSD。
    - BE 报告的磁盘类型 `storage_root_path` 同时包含 SSD 和 HDD。请注意，从 v2.3.10、v2.4.5、v2.5.4 和 v3.0 开始，当 BE 报告 `storage_root_path` 时，系统设置为 SSD 同时包含 SSD 和 HDD，并指定了属性 `storage_cooldown_time`。

  - 在以下场景中，系统会自动将该参数设置为 HDD：

    - BE 报告的磁盘类型 `storage_root_path` 仅包含 HDD。
    - 从 2.3.10、2.4.5、2.5.4 和 3.0 开始，当 BE 报告 `storage_root_path` 时，系统设置为 HDD 同时包含 SSD 和 HDD，并且 `storage_cooldown_time` 未指定该属性。

- `storage_cooldown_ttl` 或 `storage_cooldown_time`：自动存储冷却时间或时间间隔。自动存储冷却是指自动将数据从 SSD 迁移到 HDD。此功能仅在初始存储介质为 SSD 时有效。

  **参数**

  - `storage_cooldown_ttl`：本表中分区的存储自动冷却时间间隔。如果您需要在 SSD 上保留最新的分区，并在一定时间间隔后自动将旧分区冷却到 HDD，则可以使用此参数。每个分区的自动存储冷却时间是使用此参数的值加上分区的时间上限计算的。

  支持的值为 `<num> YEAR`、`<num> MONTH`、`<num> DAY` 和 `<num> HOUR`。`<num>` 是一个非负整数。默认值为 null，表示不自动执行存储冷却。

  例如，在创建表时指定值为 `"storage_cooldown_ttl"="1 DAY"`，并且指定存在范围为 `[2023-08-01 00:00:00,2023-08-02 00:00:00)` 的分区 `p20230801`。此分区的自动存储冷却时间为 `2023-08-03 00:00:00`，即 `2023-08-02 00:00:00 + 1 DAY`。如果在创建表时指定值为 `"storage_cooldown_ttl"="0 DAY"`，则此分区的自动存储冷却时间为 `2023-08-02 00:00:00`。

  - `storage_cooldown_time`：表从 SSD 冷却到 HDD 时的自动存储冷却时间（**绝对时间**）。指定的时间需要晚于当前时间。格式：“yyyy-MM-dd HH：mm：ss”。当您需要为指定分区配置不同的属性时，可以在创建表后执行 [ALTER TABLE ...添加分区或更改表...修改分区](../data-definition/ALTER_TABLE.md)。

**用法**

- 存储自动冷却相关参数对比如下：
  - `storage_cooldown_ttl`：一个表属性，用于指定表中分区的自动存储冷却时间间隔。系统在此时自动冷却分区 `the value of this parameter plus the upper time bound of the partition`。因此，在分区粒度上执行自动存储冷却，更加灵活。
  - `storage_cooldown_time`：指定此表的自动存储冷却时间（**绝对时间**）的表属性。此外，您还可以在创建表后为指定分区配置不同的属性。
  - `storage_cooldown_second`：静态 FE 参数，指定集群内所有表的自动存储冷却延迟。

- table 属性 `storage_cooldown_ttl` 或 `storage_cooldown_time` 优先于 FE 静态参数 `storage_cooldown_second`。
- 配置这些参数时，需要指定 `"storage_medium = "SSD"`.
- 如果不配置这些参数，则不会自动执行存储自动冷却。
- 执行 `SHOW PARTITIONS FROM <table_name>` 可查看每个分区的自动存储冷却时间。

**限制**

- 不支持表达式和列表分区。
- 分区列必须为日期类型。
- 不支持多个分区列。
- 不支持主键表。

**设置每个分区中每个平板电脑的副本数**

`replication_num`：每个分区中每个表的副本数。默认值： `3`。

```sql
PROPERTIES (
    "replication_num" = "<num>"
)
```

#### 为列添加布隆过滤器索引

如果引擎类型为 olap，则可以指定一个列采用布隆过滤器索引。

使用布隆过滤器索引时，存在以下限制：

- 您可以为重复键表或主键表的所有列创建布隆过滤器索引。对于聚合表或唯一键表，只能为键列创建布隆过滤器索引。
- TINYINT、FLOAT、DOUBLE 和 DECIMAL 列不支持创建布隆过滤器索引。
- 布隆过滤器索引只能提高包含 `in` 和 `=` 运算符（如 `Select xxx from table where x in {}` 和 `Select xxx from table where column = xxx`）的查询性能。此列中的离散值越多，查询就越精确。

有关详细信息，请参阅 [布隆过滤器索引](../../../using_starrocks/Bloomfilter_index.md)

```SQL
PROPERTIES (
    "bloom_filter_columns"="k1,k2,k3"
)
```

#### 使用并置连接

如果要使用并置连接属性，请在 `properties` 中指定它。

```SQL
PROPERTIES (
    "colocate_with"="table1"
)
```

#### 配置动态分区

如果要使用动态分区属性，请在属性中指定。

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

| 参数                   | 必填 | 描述                                                  |
| --------------------------- | -------- | ------------------------------------------------------------ |
| dynamic_partition.enable    | 否       | 是否启用动态分区。有效值： `TRUE` 和 `FALSE`。默认值： `TRUE`。 |
| dynamic_partition.time_unit | 是的      | 动态创建分区的时间粒度。这是一个必需参数。有效值：`DAY`、 `WEEK`和 `MONTH`。时间粒度决定了动态创建分区的后缀格式。<br/> - 如果值为 `DAY`，则动态创建分区的后缀格式为 `yyyyMMdd`。例如，分区名称后缀为 `20200321`。<br/> - 如果值为 `WEEK`，则动态创建分区的后缀格式为 `yyyy_ww`，例如 `2020_13` 表示 2020 年的第 13 周。<br/> - 如果值为 `MONTH`，则动态创建分区的后缀格式为 `yyyyMM`，例如 `202003`。 |
| dynamic_partition.start     | 不       | 动态分区的起始偏移量。此参数的值必须为负整数。这个偏移量之前的分区将根据当前日期、周或月（由 `dynamic_partition.time_unit` 确定）删除。默认值为 `Integer.MIN_VALUE`，即 -2147483648，表示历史分区不会被删除。 |
| dynamic_partition.end       | 是的      | 动态分区的结束偏移量。此参数的值必须为正整数。将提前创建从当前日、周或月到结束偏移量的分区。 |
| dynamic_partition.prefix    | 不       | 添加到动态分区名称的前缀。默认值： `p`。 |
| dynamic_partition.buckets   | 不       | 每个动态分区的存储桶数。默认值与由保留字 `BUCKETS` 确定的桶数或由 StarRocks 自动设置的桶数相同。|

#### 为配置了随机分桶的表指定存储桶大小

从 v3.2 开始，对于配置了随机分桶的表，您可以通过在创建表时使用 `PROPERTIES` 参数中的 `bucket_size` 来指定分桶大小，从而实现桶数量的按需动态增加。单位：B。

```sql
PROPERTIES (
    "bucket_size" = "1073741824"
)
```

#### 设置数据压缩算法

您可以通过在创建表时添加 `compression` 属性来指定表的数据压缩算法。

有效值 `compression` 为：

- `LZ4`：LZ4 算法。
- `ZSTD`：Zstandard 算法。
- `ZLIB`：zlib 算法。
- `SNAPPY`：Snappy 算法。

有关如何选择合适的数据压缩算法的更多信息，请参见[数据压缩](../../../table_design/data_compression.md)。

#### 设置数据加载的写入仲裁

如果您的 StarRocks 集群有多个数据副本，您可以为表设置不同的写入仲裁，即需要多少副本才能返回加载成功，然后 StarRocks 才能判断加载任务是否成功。您可以通过在创建表时添加属性来指定写入仲裁 `write_quorum` 。从 v2.5 开始支持此属性。

有效值 `write_quorum` 为：

- `MAJORITY`：默认值。当 **大多数** 数据副本返回加载成功时，StarRocks 将返回加载任务成功。否则，StarRocks 将返回加载任务失败。
- `ONE`：当 **其中一个** 数据副本返回加载成功时，StarRocks 将返回加载任务成功。否则，StarRocks 将返回加载任务失败。
- `ALL`：当 **所有** 数据副本返回加载成功时，StarRocks 将返回加载任务成功。否则，StarRocks 将返回加载任务失败。

> **注意**
>
> - 为加载设置较低的写入仲裁会增加数据不可访问甚至丢失的风险。例如，在 StarRocks 集群中，将数据加载到一个具有 1 写入仲裁的表中，该表包含两个副本，并且该数据仅成功加载到一个副本中。尽管 StarRocks 确定加载任务成功，但数据只有一个幸存的副本。如果存储加载数据的平板电脑的服务器出现故障，则这些平板电脑中的数据将无法访问。如果服务器的磁盘损坏，数据就会丢失。
> - StarRocks 只有在所有数据副本都返回状态后才会返回加载任务状态。当存在加载状态未知的副本时，StarRocks 不会返回加载任务状态。在副本中，加载超时也被视为加载失败。

#### 指定副本之间的数据写入和复制模式

如果您的 StarRocks 集群有多个数据副本，您可以指定 `replicated_storage` 中的参数 `PROPERTIES` 来配置副本之间的数据写入和复制模式。

- `true` （v3.0 及更高版本中的默认值）表示“单主复制”，这意味着数据仅写入主副本。其他副本同步主副本中的数据。此模式可显著降低数据写入多个副本导致的 CPU 成本。从 v2.5 开始支持它。
- `false` （v2.5 默认）表示“无领导复制”，即数据直接写入多个副本，不区分主副本和辅助副本。CPU 成本乘以副本数。

在大多数情况下，使用默认值可以获得更好的数据写入性能。如果要更改副本之间的数据写入和复制模式，请执行 ALTER TABLE 命令。例如：

```sql
    ALTER TABLE example_db.my_table
    SET ("replicated_storage" = "false");
```

#### 批量创建汇总

您可以在创建表时批量创建汇总。

语法：

```SQL
ROLLUP (rollup_name (column_name1, column_name2, ...)
[FROM from_index_name]
[PROPERTIES ("key"="value", ...)],...)
```

#### 为 View Delta Join 查询重写定义唯一键约束和外键约束

若要在“查看增量联接”方案中启用查询重写，必须为要在增量联接中联接的表定义“唯一键”约束 `unique_constraints` 和“外键”约束 `foreign_key_constraints` 。有关详细信息，请参阅[异步实例化视图 - 重写视图增量联接方案中的查询](../../../using_starrocks/query_rewrite_with_materialized_views.md#query-delta-join-rewrite)。

```SQL
PROPERTIES (
    "unique_constraints" = "<unique_key>[, ...]",
    "foreign_key_constraints" = "
    (<child_column>[, ...]) 
    REFERENCES 
    [catalog_name].[database_name].<parent_table_name>(<parent_column>[, ...])
    [;...]
    "
)
```

- `child_column`：表的外键。您可以定义多个 `child_column`。
- `catalog_name`：要联接的表所在的目录的名称。如果未指定此参数，则使用默认目录。
- `database_name`：要连接的表所在的数据库的名称。如果未指定此参数，则使用当前数据库。
- `parent_table_name`：要联接的表的名称。
- `parent_column`：要联接的列。它们必须是相应表的主键或唯一键。

> **注意**
>
> - `unique_constraints` 和 `foreign_key_constraints` 仅用于查询重写。将数据加载到表中时，不保证外键约束检查。您必须确保加载到表中的数据满足约束条件。
> - 默认情况下，“主键”表的主键或“唯一键”表的唯一键对应的 `unique_constraints`。您无需手动设置。
> - `child_column` 在一个表中的必须引用 `foreign_key_constraints` 另一个表中的 `unique_key`。
> - `child_column` 和 `parent_column` 的数量必须一致。
> - `child_column` 和 `parent_column` 的数据类型必须匹配。

#### 为 StarRocks 共享数据集群创建云原生表

要[使用您的 StarRocks 共享数据集群](../../../deployment/shared_data/s3.md#use-your-shared-data-starrocks-cluster)，您必须创建具有以下属性的云原生表：

```SQL
PROPERTIES (
    "datacache.enable" = "{ true | false }",
    "datacache.partition_duration" = "<string_value>",
    "enable_async_write_back" = "{ true | false }"
)
```

- `datacache.enable`：是否启用本地磁盘缓存。默认值： `true`。

  - 当此属性设置为 `true` 时，要加载的数据将同时写入对象存储和本地磁盘（作为查询加速的缓存）。
  - 当此属性设置为 `false` 时，数据仅加载到对象存储中。

  > **注意**
  >
  > 要启用本地磁盘缓存，必须在 BE 配置项中指定磁盘的目录 `storage_root_path`。更多信息，请参见 [BE配置项](../../../administration/BE_configuration.md#be-configuration-items)。

- `datacache.partition_duration`：热点数据的有效期。启用本地磁盘缓存后，所有数据都会加载到缓存中。当缓存已满时，StarRocks 会从缓存中删除最近使用较少的数据。当查询需要扫描已删除的数据时，StarRocks 会检查数据是否在有效期内。如果数据在持续时间内，StarRocks 会再次将数据加载到缓存中。如果数据不在持续时间内，StarRocks 不会将其加载到缓存中。此属性是一个字符串值，可以使用以下单位指定：`YEAR`、`MONTH`、`DAY`、`HOUR`，例如 `7 DAY` 和 `12 HOUR`。如果未指定，则所有数据都缓存为热数据。

  > **注意**
  >
  > 仅当 `datacache.enable` 设置为 `true` 时，此属性才可用。

- `enable_async_write_back`：是否允许数据异步写入对象存储。默认值： `false`。
  - 当此属性设置为 `true` 时，只要数据写入本地磁盘缓存，数据就会异步写入对象存储，加载任务就会立即返回成功。这样可以提高加载性能，但也会在潜在的系统故障下危及数据可靠性。
  - 当此属性设置为 `false` 时，只有在数据写入对象存储和本地磁盘缓存后，加载任务才会返回成功。这样可以保证更高的可用性，但会导致较低的加载性能。

#### 设置快速架构演进

`fast_schema_evolution`：是否启用表的快速架构演进。有效值为 `TRUE` 或 `FALSE`（默认值）。启用快速架构演进可以加快架构更改的速度，并在添加或删除列时减少资源使用量。目前，此属性只能在创建表时启用，并且无法在创建表后使用 [ALTER TABLE](../../sql-statements/data-definition/ALTER_TABLE.md) 进行修改。该参数自 v3.2.0 版本开始支持。
  > **注意**
  >
  > - StarRocks 共享数据集群不支持该参数。
  > - 如果需要在集群级别配置快速架构演进，例如在 StarRocks 集群内禁用快速架构演进，可以设置 FE 动态参数 [`enable_fast_schema_evolution`](../../../administration/FE_configuration.md#enable_fast_schema_evolution)。

## 示例

### 创建使用哈希桶和列式存储的聚合表

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

### 创建聚合表并设置存储介质和冷却时间

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

### 创建使用范围分区、哈希分桶和列存储的重复键表，并设置存储介质和冷却时间

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

注意：

此语句将创建三个数据分区：

```SQL
( {    MIN     },   {"2014-01-01"} )
[ {"2014-01-01"},   {"2014-06-01"} )
[ {"2014-06-01"},   {"2014-12-01"} )
```

超出这些范围的数据将不会加载。

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

### 创建 MySQL 外部表

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

### 创建包含 HLL 列的表

```SQL
CREATE TABLE example_db.example_table
(
    k1 TINYINT,
    k2 DECIMAL(10, 2) DEFAULT "10.5",
    v1 HLL HLL_UNION,
    v2 HLL HLL_UNION
)
ENGINE=olap
AGGREGATE KEY(k1, k2)
DISTRIBUTED BY HASH(k1)
PROPERTIES ("storage_type"="column");
```

### 创建包含 BITMAP_UNION 聚合类型的表

`v1` 和 `v2` 列的原始数据类型必须为 TINYINT、SMALLINT 或 INT。

```SQL
CREATE TABLE example_db.example_table
(
    k1 TINYINT,
    k2 DECIMAL(10, 2) DEFAULT "10.5",
    v1 BITMAP BITMAP_UNION,
    v2 BITMAP BITMAP_UNION
)
ENGINE=olap
AGGREGATE KEY(k1, k2)
DISTRIBUTED BY HASH(k1)
PROPERTIES ("storage_type"="column");
```

### 创建两个支持共置联接的表

```SQL
CREATE TABLE `t1` 
(
     `id` int(11) COMMENT "",
    `value` varchar(8) COMMENT ""
) 
ENGINE=OLAP
DUPLICATE KEY(`id`)
DISTRIBUTED BY HASH(`id`)
PROPERTIES 
(
    "colocate_with" = "t1"
);

CREATE TABLE `t2` 
(
    `id` int(11) COMMENT "",
    `value` varchar(8) COMMENT ""
) 
ENGINE=OLAP
DUPLICATE KEY(`id`)
DISTRIBUTED BY HASH(`id`)
PROPERTIES 
(
    "colocate_with" = "t1"
);
```

### 创建具有位图索引的表

```SQL
CREATE TABLE example_db.table_hash
(
    k1 TINYINT,
    k2 DECIMAL(10, 2) DEFAULT "10.5",
    v1 CHAR(10) REPLACE,
    v2 INT SUM,
    INDEX k1_idx (k1) USING BITMAP COMMENT 'xxxxxx'
)
ENGINE=olap
AGGREGATE KEY(k1, k2)
COMMENT "my first starrocks table"
DISTRIBUTED BY HASH(k1)
PROPERTIES ("storage_type"="column");
```

### 创建动态分区表

在 FE 配置中必须启用动态分区功能（“dynamic_partition.enable” = “true”）。有关详细信息，请参阅 [配置动态分区](#configure-dynamic-partitions)。

此示例为接下来的三天创建分区，并删除三天前创建的分区。例如，如果今天是 2020-01-08，则将创建具有以下名称的分区：p20200108、p20200109、p20200110、p20200111，其范围为：

```plaintext
[types: [DATE]; keys: [2020-01-08]; ‥types: [DATE]; keys: [2020-01-09]; )
[types: [DATE]; keys: [2020-01-09]; ‥types: [DATE]; keys: [2020-01-10]; )
[types: [DATE]; keys: [2020-01-10]; ‥types: [DATE]; keys: [2020-01-11]; )
[types: [DATE]; keys: [2020-01-11]; ‥types: [DATE]; keys: [2020-01-12]; )
```

```SQL
CREATE TABLE example_db.dynamic_partition
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
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "DAY",
    "dynamic_partition.start" = "-3",
    "dynamic_partition.end" = "3",
    "dynamic_partition.prefix" = "p",
    "dynamic_partition.buckets" = "10"
);
```

### 创建一个表，其中批量创建多个分区，并将整数类型的列指定为分区列

在以下示例中，分区列 `datekey` 的类型为 INT。所有分区仅由一个简单的分区子句创建`START ("1") END ("5") EVERY (1)`。所有分区的范围从  开始`1`，到 `5`结束，分区粒度为 `1`：
  > **注意**
  >
  > START（）  和 ** END（） 中的分区列值**需要用引号括起来，而 EVERY**（）** 中的分区粒度不需要用引号括起来**。**

  ```SQL
  CREATE TABLE site_access (
      datekey INT,
      site_id INT,
      city_code SMALLINT,
      user_name VARCHAR(32),
      pv BIGINT DEFAULT '0'
  )
  ENGINE=olap
  DUPLICATE KEY(datekey, site_id, city_code, user_name)
  PARTITION BY RANGE (datekey) (START ("1") END ("5") EVERY (1)
  )
  DISTRIBUTED BY HASH(site_id)
  PROPERTIES ("replication_num" = "3");
  ```
### 创建 Hive 外部表

在创建 Hive 外部表之前，您必须已创建 Hive 资源和数据库。有关详细信息，请参阅 [外部表](../../../data_source/External_table.md#deprecated-hive-external-table)。

```SQL
CREATE EXTERNAL TABLE example_db.table_hive
(
    k1 TINYINT,
    k2 VARCHAR(50),
    v INT
)
ENGINE=hive
PROPERTIES
(
    "resource" = "hive0",
    "database" = "hive_db_name",
    "table" = "hive_table_name"
);
```

### 创建主键表并指定排序键

假设您需要实时分析用户行为，从用户地址和上次活动时间等维度。在创建表时，您可以将 `user_id` 列定义为主键，并将 `address` 和 `last_active` 列的组合定义为排序键。

```SQL
create table users (
    user_id bigint NOT NULL,
    name string NOT NULL,
    email string NULL,
    address string NULL,
    age tinyint NULL,
    sex tinyint NULL,
    last_active datetime,
    property0 tinyint NOT NULL,
    property1 tinyint NOT NULL,
    property2 tinyint NOT NULL,
    property3 tinyint NOT NULL
) 
PRIMARY KEY (`user_id`)
DISTRIBUTED BY HASH(`user_id`)
ORDER BY(`address`,`last_active`)
PROPERTIES(
    "replication_num" = "3",
    "enable_persistent_index" = "true"
);
```

## 引用

- [显示创建表](../data-manipulation/SHOW_CREATE_TABLE.md)
- [显示表格](../data-manipulation/SHOW_TABLES.md)
- [使用](USE.md)
- [更改表](ALTER_TABLE.md)
- [删除表](DROP_TABLE.md)