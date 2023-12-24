---
displayed_sidebar: English
---

# 更改表

## 描述

修改现有表，包括：

- [重命名表、分区、索引](#rename)
- [修改表注释](#alter-table-comment-from-v31)
- [修改分区（添加/删除分区和修改分区属性）](#modify-partition)
- [修改分桶方式和分桶数量](#modify-the-bucketing-method-and-number-of-buckets-from-v32)
- [修改列（添加/删除列和更改列的顺序）](#modify-columns-adddelete-columns-change-the-order-of-columns)
- [创建/删除汇总索引](#modify-rollup-index)
- [修改位图索引](#modify-bitmap-indexes)
- [修改表属性](#modify-table-properties)
- [原子交换](#swap)
- [手动数据版本压缩](#manual-compaction-from-31)

> **注意**
>
> 此操作需要对目标表具有 ALTER 权限。

## 语法

```SQL
ALTER TABLE [<db_name>.]<tbl_name>
alter_clause1[, alter_clause2, ...]
```

`alter_clause` 可以包含以下操作：重命名、注释、分区、存储桶、列、汇总索引、位图索引、表属性、交换和压缩。

- 重命名：重命名表、汇总索引或分区。 **请注意，无法修改列名。**
- 注释：修改表注释（从 **v3.1 开始支持**）。
- 分区：修改分区属性、删除分区或添加分区。
- 存储桶：修改存储桶方式和存储桶数量。
- 列：添加、删除或重新排序列，或修改列类型。
- 汇总索引：创建或删除汇总索引。
- 位图索引：修改索引（仅能修改位图索引）。
- 交换：两个表的原子交换。
- 压缩：执行手动压缩以合并加载数据的版本（从 **v3.1 开始支持**）。

:::note

- 不能在一个 ALTER TABLE 语句中对分区、列和汇总索引执行操作。
- 存储桶、列和汇总索引的操作是异步操作。提交任务后，将立即返回成功消息。您可以执行[SHOW ALTER TABLE](../data-manipulation/SHOW_ALTER.md)命令查看操作进度，执行CANCEL ALTER[ TABLE命令取消操作。](../data-definition/CANCEL_ALTER_TABLE.md) 
- 重命名、注释、分区、位图索引和交换操作是同步操作，返回命令表示执行完成。
:::

### 重命名

重命名支持修改表名、汇总索引和分区名。

#### 重命名表

```sql
ALTER TABLE <tbl_name> RENAME <new_tbl_name>
```

#### 重命名汇总索引

```sql
ALTER TABLE [<db_name>.]<tbl_name>
RENAME ROLLUP <old_rollup_name> <new_rollup_name>
```

#### 重命名分区

```sql
ALTER TABLE [<db_name>.]<tbl_name>
RENAME PARTITION <old_partition_name> <new_partition_name>
```

### 修改表注释（从 v3.1 开始）

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name> COMMENT = "<new table comment>";
```

### 修改分区

#### 添加分区

语法：

```SQL
ALTER TABLE [<db_name>.]<tbl_name> 
ADD PARTITION [IF NOT EXISTS] <partition_name>
partition_desc ["key"="value"]
[DISTRIBUTED BY HASH (k1[,k2 ...]) [BUCKETS num]];
```

注意：

1. Partition_desc支持以下两种表达式：

    ```plain
    VALUES LESS THAN [MAXVALUE|("value1", ...)]
    VALUES ("value1", ...), ("value1", ...)
    ```

2. partition 是左-闭-右-开间隔。如果用户只指定了右边界，系统将自动确定左边界。
3. 如果未指定 bucket 模式，则自动使用内置表使用的 bucket 方法。
4. 如果指定了桶模式，则只能修改桶号，不能修改桶模式或桶列。
5. 用户可以在 中设置分区的某些属性`["key"="value"]`。[ 有关详细信息，请参见 ](CREATE_TABLE.md)CREATE TABLE。

#### 删除分区

语法：

```sql
-- Before 2.0
ALTER TABLE [<db_name>.]<tbl_name>
DROP PARTITION [IF EXISTS | FORCE] <partition_name>
-- 2.0 or later
ALTER TABLE [<db_name>.]<tbl_name>
DROP PARTITION [IF EXISTS] <partition_name> [FORCE]
```

注意：

1. 为分区表保留至少一个分区。
2. 执行 DROP PARTITION 一段时间后，可以通过 RECOVER 语句恢复删除的分区。有关详细信息，请参见 RECOVER 语句。
3. 如果执行 DROP PARTITION FORCE，则该分区将被直接删除，如果不检查分区上是否有任何未完成的活动，则无法恢复。因此，通常不建议执行此操作。

#### 添加临时分区

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name> 
ADD TEMPORARY PARTITION [IF NOT EXISTS] <partition_name>
partition_desc ["key"="value"]
[DISTRIBUTED BY HASH (k1[,k2 ...]) [BUCKETS num]]
```

#### 使用临时分区替换当前分区

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
REPLACE PARTITION <partition_name>
partition_desc ["key"="value"]
WITH TEMPORARY PARTITION
partition_desc ["key"="value"]
[PROPERTIES ("key"="value", ...)]
```

#### 删除临时分区

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP TEMPORARY PARTITION <partition_name>
```

#### 修改分区属性

**语法**

```sql
ALTER TABLE [<db_name>.]<tbl_name>
    MODIFY PARTITION { <partition_name> | partition_name_list | (*) }
        SET ("key" = "value", ...);
```

**用法**

- 可以修改分区的以下属性：

  - storage_medium
  - storage_cooldown_ttl或storage_cooldown_time
  - replication_num

- 对于只有一个分区的表，分区名与表名相同。如果表被划分为多个分区，则可以用来 `(*)`修改所有分区的属性，这样更方便。

- 执行`SHOW PARTITIONS FROM <tbl_name>`以查看修改后的分区属性。

### 修改分桶方式和分桶数量（从 v3.2 开始）

语法：

```SQL
ALTER TABLE [<db_name>.]<table_name>
[ partition_names ]
[ distribution_desc ]

partition_names ::= 
    (PARTITION | PARTITIONS) ( <partition_name> [, <partition_name> ...] )

distribution_desc ::=
    DISTRIBUTED BY RANDOM [ BUCKETS <num> ] |
    DISTRIBUTED BY HASH ( <column_name> [, <column_name> ...] ) [ BUCKETS <num> ]
```

例：

例如，原来的表是 Duplicate Key 表，其中使用了 hash bucket，桶的数量由 StarRocks 自动设置。

```SQL
CREATE TABLE IF NOT EXISTS details (
    event_time DATETIME NOT NULL COMMENT "datetime of event",
    event_type INT NOT NULL COMMENT "type of event",
    user_id INT COMMENT "id of user",
    device_code INT COMMENT "device code",
    channel INT COMMENT ""
)
DUPLICATE KEY(event_time, event_type)
PARTITION BY date_trunc('day', event_time)
DISTRIBUTED BY HASH(user_id);

-- Insert data of several days
INSERT INTO details (event_time, event_type, user_id, device_code, channel) VALUES
-- Data of November 26th
('2023-11-26 08:00:00', 1, 101, 12345, 2),
('2023-11-26 09:15:00', 2, 102, 54321, 3),
('2023-11-26 10:30:00', 1, 103, 98765, 1),
-- Data of November 27th
('2023-11-27 08:30:00', 1, 104, 11111, 2),
('2023-11-27 09:45:00', 2, 105, 22222, 3),
('2023-11-27 11:00:00', 1, 106, 33333, 1),
-- Data of November 28th
('2023-11-28 08:00:00', 1, 107, 44444, 2),
('2023-11-28 09:15:00', 2, 108, 55555, 3),
('2023-11-28 10:30:00', 1, 109, 66666, 1);
```

#### 仅修改分桶方式

> **注意**
>
> - 修改将应用于表中的所有分区，不能仅应用于特定分区。
> - 虽然只需要修改分桶方式，但仍然需要在命令中使用指定分桶的数量 `BUCKETS <num>`。如果 `BUCKETS <num>` 未指定，则表示 Bucket 数量由 StarRocks 自动决定。

- 将 Bucketing 方式由哈希桶修改为随机桶，桶数仍由 StarRocks 自动设置。

  ```SQL
  ALTER TABLE details DISTRIBUTED BY RANDOM;
  ```

- 将哈希分桶的键由 `event_time, event_type` 修改为 `user_id, event_time`。桶数仍由 StarRocks 自动设置。

  ```SQL
  ALTER TABLE details DISTRIBUTED BY HASH(user_id, event_time);
  ```

#### 仅修改存储桶数量

> **注意**
>
> 虽然只需要修改桶的数量，但仍然需要在命令中指定桶的方式，例如 `HASH(user_id)`.

- 将所有分区的 Bucket 数量从 StarRocks 自动设置为 10 个。

  ```SQL
  ALTER TABLE details DISTRIBUTED BY HASH(user_id) BUCKETS 10;
  ```

- 将指定分区的 Bucket 数量从 StarRocks 自动设置为 15 个。

  ```SQL
  ALTER TABLE details PARTITIONS (p20231127, p20231128) DISTRIBUTED BY HASH(user_id) BUCKETS 15 ;
  ```

  > **注意**
  >
  > 可以通过执行 来查看分区名称 `SHOW PARTITIONS FROM <table_name>;`。

#### 修改分桶方式和分桶数量

> **注意**
>
> 修改将应用于表中的所有分区，不能仅应用于特定分区。


- 将分桶方法从哈希分桶修改为随机分桶，并将桶数从 StarRocks 自动设置改为 10 个。

   ```SQL
   ALTER TABLE details DISTRIBUTED BY RANDOM BUCKETS 10;
   ```

- 修改哈希分桶的密钥，并将桶数从 StarRocks 自动设置改为 10 个。用于哈希分桶的密钥已从原始的 `event_time, event_type` 修改为 `user_id, event_time`。桶数已从 StarRocks 自动设置修改为 10 个。

  ```SQL
  ALTER TABLE details DISTRIBUTED BY HASH(user_id, event_time) BUCKETS 10;
  ``

### 修改列（添加/删除列，更改列的顺序）

#### 在指定索引的指定位置添加列

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ADD COLUMN column_name column_type [KEY | agg_type] [DEFAULT "default_value"]
[AFTER column_name|FIRST]
[TO rollup_index_name]
[PROPERTIES ("key"="value", ...)]
```

注意：

1. 如果向聚合表添加值列，则需要指定 agg_type。
2. 如果向非聚合表（如重复键表）添加键列，则需要指定 KEY 关键字。
3. 不能将已存在于基本索引中的列添加到汇总索引中。（如果需要，可以重新创建汇总索引。）

#### 向指定索引添加多列

语法：

- 添加多列

  ```sql
  ALTER TABLE [<db_name>.]<tbl_name>
  ADD COLUMN (column_name1 column_type [KEY | agg_type] DEFAULT "default_value", ...)
  [TO rollup_index_name]
  [PROPERTIES ("key"="value", ...)]
  ```

- 添加多个列并使用 AFTER 指定添加的列的位置

  ```sql
  ALTER TABLE [<db_name>.]<tbl_name>
  ADD COLUMN column_name1 column_type [KEY | agg_type] DEFAULT "default_value" AFTER column_name,
  ADD COLUMN column_name2 column_type [KEY | agg_type] DEFAULT "default_value" AFTER column_name
  [, ...]
  [TO rollup_index_name]
  [PROPERTIES ("key"="value", ...)]
  ```

注意：

1. 如果向聚合表添加值列，则需要指定 `agg_type`。
2. 如果向非聚合表添加键列，则需要指定 KEY 关键字。
3. 不能将已存在于基本索引中的列添加到汇总索引中。（如果需要，可以创建另一个汇总索引。）

#### 添加生成的列（从 v3.1 开始）

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ADD COLUMN col_name data_type [NULL] AS generation_expr [COMMENT 'string']
```

您可以添加生成的列并指定其表达式。[生成的列](../generated_columns.md) 可用于预计算和存储表达式的结果，从而显著加快了使用相同复杂表达式的查询速度。从 v3.1 开始，StarRocks 支持生成列。

#### 从指定索引中删除列

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP COLUMN column_name
[FROM rollup_index_name];
```

注意：

1. 不能删除分区列。
2. 如果从基本索引中删除该列，则在汇总索引中包含该列时，该列也将被删除。

#### 修改指定索引的列类型和列位置

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
MODIFY COLUMN column_name column_type [KEY | agg_type] [NULL | NOT NULL] [DEFAULT "default_value"]
[AFTER column_name|FIRST]
[FROM rollup_index_name]
[PROPERTIES ("key"="value", ...)]
```

注意：

1. 如果修改聚合模型中的值列，则需要指定 agg_type。
2. 如果修改非聚合模型中的键列，则需要指定 KEY 关键字。
3. 只能修改列的类型。列的其他属性将保持当前状态。（即其他属性需要根据原始属性显式写在语句中，参见 [列](#column) 部分）。
4. 分区列不能修改。
5. 目前支持以下类型的转换（准确性损失由用户保证）。

   - 将 TINYINT/SMALLINT/INT/BIGINT 转换为 TINYINT/SMALLINT/INT/BIGINT/DOUBLE。
   - 将 TINTINT/SMALLINT/INT/BIGINT/LARGEINT/FLOAT/DOUBLE/DECIMAL 转换为 VARCHAR。VARCHAR 支持修改最大长度。
   - 将 VARCHAR 转换为 TINTINT/SMALLINT/INT/BIGINT/LARGEINT/FLOAT/DOUBLE。
   - 将 VARCHAR 转换为 DATE（目前支持六种格式：“%Y-%m-%d”、“%y-%m-%d”、“%Y%m%d”、“%y%m%d”、“%Y/%m/%d”、“%y/%m/%d”）
   - 将 DATETIME 转换为 DATE（仅保留年-月-日信息，即 `2019-12-09 21:47:05` `<-->` `2019-12-09`）
   - 将 DATE 转换为 DATETIME（将小时、分钟、秒设置为零，例如： `2019-12-09` `<-->` `2019-12-09 00:00:00`）
   - 将 FLOAT 转换为 DOUBLE
   - 将 INT 转换为 DATE（如果 INT 数据转换失败，则原始数据保持不变）

6. 不支持从 NULL 到 NOT NULL 的转换。

#### 对指定索引的列重新排序

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ORDER BY (column_name1, column_name2, ...)
[FROM rollup_index_name]
[PROPERTIES ("key"="value", ...)]
```

注意：

- 必须写入索引中的所有列。
- 值列在键列之后。

#### 修改主键表中排序键的列

<!--Supported Versions-->

语法：

```SQL
ALTER TABLE [<db_name>.]<table_name>
[ order_desc ]

order_desc ::=
    ORDER BY <column_name> [, <column_name> ...]
```

例：

例如，原始表是排序键和主键耦合的主键表，即 `dt, order_id`.

```SQL
create table orders (
    dt date NOT NULL,
    order_id bigint NOT NULL,
    user_id int NOT NULL,
    merchant_id int NOT NULL,
    good_id int NOT NULL,
    good_name string NOT NULL,
    price int NOT NULL,
    cnt int NOT NULL,
    revenue int NOT NULL,
    state tinyint NOT NULL
) PRIMARY KEY (dt, order_id)
PARTITION BY date_trunc('day', dt)
DISTRIBUTED BY HASH(order_id);
```

将排序键与主键分离，并将排序键修改为 `dt, revenue, state`.

```SQL
ALTER TABLE orders ORDER BY (dt, revenue, state);
```

### 修改汇总索引

#### 创建汇总索引

语法：

```SQL
ALTER TABLE [<db_name>.]<tbl_name> 
ADD ROLLUP rollup_name (column_name1, column_name2, ...)
[FROM from_index_name]
[PROPERTIES ("key"="value", ...)]
```

PROPERTIES：支持设置超时时间，默认超时时间为一天。

例：

```SQL
ALTER TABLE [<db_name>.]<tbl_name> 
ADD ROLLUP r1(col1,col2) from r0;
```

#### 批量创建汇总索引

语法：

```SQL
ALTER TABLE [<db_name>.]<tbl_name>
ADD ROLLUP [rollup_name (column_name1, column_name2, ...)
[FROM from_index_name]
[PROPERTIES ("key"="value", ...)],...];
```

例：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ADD ROLLUP r1(col1,col2) from r0, r2(col3,col4) from r0;
```

注意：

1. 如果未指定 from_index_name，则默认从基索引创建。
2. 汇总表中的列必须是 from_index 中的现有列。
3. 在属性中，用户可以指定存储格式。有关详细信息，请参见 CREATE TABLE。

#### 删除汇总索引

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP ROLLUP rollup_name [PROPERTIES ("key"="value", ...)];
```

例：

```sql
ALTER TABLE [<db_name>.]<tbl_name> DROP ROLLUP r1;
```

#### 批量删除汇总索引

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP ROLLUP [rollup_name [PROPERTIES ("key"="value", ...)],...];
```

例：

```sql
ALTER TABLE [<db_name>.]<tbl_name> DROP ROLLUP r1, r2;
```

注意：不能删除基索引。

### 修改位图索引

位图索引支持以下修改：

#### 创建位图索引

语法：

```sql
 ALTER TABLE [<db_name>.]<tbl_name>
ADD INDEX index_name (column [, ...],) [USING BITMAP] [COMMENT 'balabala'];
```

注意：

```plain text
1. 位图索引仅支持当前版本。
2. 仅在单个列中创建 BITMAP 索引。
```

#### 删除位图索引

语法：

```sql
DROP INDEX index_name;
```

### 修改表属性

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
SET ("key" = "value",...)
```

目前，StarRocks 支持修改以下表属性：

- `replication_num`
- `default.replication_num`
- `storage_cooldown_ttl`
- `storage_cooldown_time`
- 动态分区相关属性
- `enable_persistent_index`
- `bloom_filter_columns`
- `colocate_with`
- `bucket_size` （从 3.2 开始支持）

注意：
您还可以通过合并到上述列操作中来修改属性。请参阅以下示例[](#examples)。

### 交换

Swap 支持两个表的原子交换。

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
SWAP WITH <tbl_name>;
```

### 手动压实（从 3.1 开始）

StarRocks 使用压缩机制来合并不同版本的加载数据。此功能可以将小文件合并为大文件，有效提高查询性能。

在 v3.1 之前，压缩有两种方式：

- 系统自动压实：在后台的 BE 级别执行压实。用户无法指定用于压缩的数据库或表。
- 用户可以通过调用HTTP接口进行压实。
从 v3.1 开始，StarRocks 为用户提供了 SQL 接口，以便通过运行 SQL 命令手动执行压缩。他们可以选择特定的表或分区进行压缩。这样可以更灵活地控制压缩过程。

语法：

```sql
-- 对整个表执行压缩。
ALTER TABLE <tbl_name> COMPACT

-- 对单个分区执行压缩。
ALTER TABLE <tbl_name> COMPACT <partition_name>

-- 对多个分区执行压缩。
ALTER TABLE <tbl_name> COMPACT (<partition1_name>[,<partition2_name>,...])

-- 执行累积压缩。
ALTER TABLE <tbl_name> CUMULATIVE COMPACT (<partition1_name>[,<partition2_name>,...])

-- 执行基础压缩。
ALTER TABLE <tbl_name> BASE COMPACT (<partition1_name>[,<partition2_name>,...])
```

 `information_schema` 数据库中的 `be_compactions` 表记录了压缩结果。您可以运行 `SELECT * FROM information_schema.be_compactions;` 查询压缩后的数据版本。

## 示例

### 表

1. 修改表的默认副本数，该副本数用作新添加分区的默认副本数。

    ```sql
    ALTER TABLE example_db.my_table
    SET ("default.replication_num" = "2");
    ```

2. 修改单分区表的实际副本数。

    ```sql
    ALTER TABLE example_db.my_table
    SET ("replication_num" = "3");
    ```

3. 修改副本间的数据写入和复制模式。

    ```sql
    ALTER TABLE example_db.my_table
    SET ("replicated_storage" = "false");
    ```

   本示例将副本间的数据写入和复制模式设置为“无领导复制”，即数据同时写入多个副本，不区分主副本和辅助副本。有关详细信息，请参阅 [CREATE TABLE](CREATE_TABLE.md) 中的 `replicated_storage` 参数。

### 分区

1. 添加分区并使用默认的分桶模式。现有分区为 [MIN， 2013-01-01]。添加的分区为 [2013-01-01， 2014-01-01]。

    ```sql
    ALTER TABLE example_db.my_table
    ADD PARTITION p1 VALUES LESS THAN ("2014-01-01");
    ```

2. 添加分区并使用新数量的存储桶。

    ```sql
    ALTER TABLE example_db.my_table
    ADD PARTITION p1 VALUES LESS THAN ("2015-01-01")
    DISTRIBUTED BY HASH(k1);
    ```

3. 添加分区并使用新的副本数。

    ```sql
    ALTER TABLE example_db.my_table
    ADD PARTITION p1 VALUES LESS THAN ("2015-01-01")
    ("replication_num"="1");
    ```

4. 更改分区的副本数。

    ```sql
    ALTER TABLE example_db.my_table
    MODIFY PARTITION p1 SET("replication_num"="1");
    ```

5. 批量更改指定分区的副本数。

    ```sql
    ALTER TABLE example_db.my_table
    MODIFY PARTITION (p1, p2, p4) SET("replication_num"="1");
    ```

6. 批量更改所有分区的存储介质。

    ```sql
    ALTER TABLE example_db.my_table
    MODIFY PARTITION (*) SET("storage_medium"="HDD");
    ```

7. 删除分区。

    ```sql
    ALTER TABLE example_db.my_table
    DROP PARTITION p1;
    ```

8. 添加具有上限和下限的分区。

    ```sql
    ALTER TABLE example_db.my_table
    ADD PARTITION p1 VALUES [("2014-01-01"), ("2014-02-01"));
    ```

### 汇总索引

1. 基于基本索引（k1，k2，k3，v1，v2）创建汇总索引 `example_rollup_index` 。使用基于列的存储。

    ```sql
    ALTER TABLE example_db.my_table
    ADD ROLLUP example_rollup_index(k1, k3, v1, v2)
    PROPERTIES("storage_type"="column");
    ```

2.  基于 `example_rollup_index2`创建`example_rollup_index(k1,k3,v1,v2)`索引。

    ```sql
    ALTER TABLE example_db.my_table
    ADD ROLLUP example_rollup_index2 (k1, v1)
    FROM example_rollup_index;
    ```

3.  `example_rollup_index3`基于基本索引（k1、k2、k3、v1）创建索引。汇总超时时间设置为 1 小时。

    ```sql
    ALTER TABLE example_db.my_table
    ADD ROLLUP example_rollup_index3(k1, k3, v1)
    PROPERTIES("storage_type"="column", "timeout" = "3600");
    ```

4. 删除索引 `example_rollup_index2`。

    ```sql
    ALTER TABLE example_db.my_table
    DROP ROLLUP example_rollup_index2;
    ```

### 列

1. 在 的列之后添加一个键`new_col`列（非聚合 `col1` 列`example_rollup_index`）。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN new_col INT KEY DEFAULT "0" AFTER col1
    TO example_rollup_index;
    ```

2. 在 的列后面添加一个值`new_col`列（非聚合 `col1` 列`example_rollup_index`）。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN new_col INT DEFAULT "0" AFTER col1
    TO example_rollup_index;
    ```

3. 在 的列后面添加一个键`new_col`列（聚合列）。 `col1` `example_rollup_index`

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN new_col INT DEFAULT "0" AFTER col1
    TO example_rollup_index;
    ```

4. 在 的列后添加一个值`new_col SUM`列（聚合列）。 `col1` `example_rollup_index`

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN new_col INT SUM DEFAULT "0" AFTER col1
    TO example_rollup_index;
    ```

5. 向  （aggregate）`example_rollup_index` 添加多个列。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN (col1 INT DEFAULT "1", col2 FLOAT SUM DEFAULT "2.3")
    TO example_rollup_index;
    ```

6. 将多个列添加到 `example_rollup_index` （聚合）并使用 指定所添加列的位置 `AFTER`。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN col1 INT DEFAULT "1" AFTER `k1`,
    ADD COLUMN col2 FLOAT SUM AFTER `v2`,
    TO example_rollup_index;
    ```

7. 从 中删除一列 `example_rollup_index`。

    ```sql
    ALTER TABLE example_db.my_table
    DROP COLUMN col2
    FROM example_rollup_index;
    ```

8. 将基索引的col1的列类型修改为BIGINT，放在后面`col2`。

    ```sql
    ALTER TABLE example_db.my_table
    MODIFY COLUMN col1 BIGINT DEFAULT "1" AFTER col2;
    ```

9. 将 `val1` 基索引列的最大长度修改为 64。原始长度为 32。

    ```sql
    ALTER TABLE example_db.my_table
    MODIFY COLUMN val1 VARCHAR(64) REPLACE DEFAULT "abc";
    ```

10. 对 中的列重新排序 `example_rollup_index`。原始列顺序为 k1、k2、k3、v1、v2。

    ```sql
    ALTER TABLE example_db.my_table
    ORDER BY (k3,k1,k2,v2,v1)
    FROM example_rollup_index;
    ```

11. 一次执行两个操作（ADD COLUMN 和 ORDER BY）。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN v2 INT MAX DEFAULT "0" AFTER k2 TO example_rollup_index,
    ORDER BY (k3,k1,k2,v2,v1) FROM example_rollup_index;
    ```

12. 更改表的 bloomfilter 列。

     ```sql
     ALTER TABLE example_db.my_table
     SET ("bloom_filter_columns"="k1,k2,k3");
     ```

     该操作也可以合并到上面的列操作中（注意多个子句的语法略有不同）。

     ```sql
     ALTER TABLE example_db.my_table
     DROP COLUMN col2
     PROPERTIES ("bloom_filter_columns"="k1,k2,k3");
     ```

### 表属性

1. 更改表的 Colocate 属性。

     ```sql
     ALTER TABLE example_db.my_table
     SET ("colocate_with" = "t1");
     ```

2. 更改表的动态分区属性。

     ```sql
     ALTER TABLE example_db.my_table
     SET ("dynamic_partition.enable" = "false");
     ```

     如果需要将动态分区属性添加到未配置动态分区属性的表中，则需要指定所有动态分区属性。

     ```sql
     ALTER TABLE example_db.my_table
     SET (
         "dynamic_partition.enable" = "true",
         "dynamic_partition.time_unit" = "DAY",
         "dynamic_partition.end" = "3",
         "dynamic_partition.prefix" = "p",
         "dynamic_partition.buckets" = "32"
         );
     ```

### 重命名

1. 重命名 `table1` 为 `table2`.

    ```sql
    ALTER TABLE table1 RENAME table2;
    ```

2. 将 的汇总索引重命名 `rollup1` 为 `example_table` `rollup2`。

    ```sql
    ALTER TABLE example_table RENAME ROLLUP rollup1 rollup2;
    ```

3. 将 的分区重命名 `p1` `example_table` 为 `p2`。

    ```sql
    ALTER TABLE example_table RENAME PARTITION p1 p2;
    ```

### 位图索引

1. 为 中的列创建位图索引 `siteid` `table1`。

    ```sql
    ALTER TABLE table1
    ADD INDEX index_1 (siteid) [USING BITMAP] COMMENT 'balabala';
    ```

2. 删除 中列的位图索引 `siteid` `table1`。

    ```sql
    ALTER TABLE table1
    DROP INDEX index_1;
    ```

### 交换

 和 `table1`之间的`table2`原子交换。

```sql
ALTER TABLE table1 SWAP WITH table2
```

### 手动压实

```sql
CREATE TABLE compaction_test( 
    event_day DATE,
    pv BIGINT)
DUPLICATE KEY(event_day)
PARTITION BY date_trunc('month', event_day)
DISTRIBUTED BY HASH(event_day) BUCKETS 8
PROPERTIES("replication_num" = "3");

INSERT INTO compaction_test VALUES

('2023-02-14', 2),
('2033-03-01',2);
{'label':'insert_734648fa-c878-11ed-90d6-00163e0dcbfc', 'status':'VISIBLE', 'txnId':'5008'}

向 compaction_test 表中插入数值
('2023-02-14', 2),('2033-03-01',2);
{'label':'insert_85c95c1b-c878-11ed-90d6-00163e0dcbfc', 'status':'VISIBLE', 'txnId':'5009'}

ALTER TABLE compaction_test 进行压缩;

ALTER TABLE compaction_test 进行 p203303 压缩;

ALTER TABLE compaction_test 进行 (p202302,p203303) 压缩;

ALTER TABLE compaction_test 累积压缩 (p202302,p203303);

ALTER TABLE compaction_test 基础压缩 (p202302,p203303);
```

## 参考

- [创建表](./CREATE_TABLE.md)
- [显示创建表](../data-manipulation/SHOW_CREATE_TABLE.md)
- [显示表格](../data-manipulation/SHOW_TABLES.md)
- [显示更改表](../data-manipulation/SHOW_ALTER.md)
- [删除表](./DROP_TABLE.md)