---
displayed_sidebar: English
---

# 修改表

## 描述

修改现有表，包括：

- [重命名表、分区、索引](#rename)
- [修改表注释（从 v3.1 版本开始）](#alter-table-comment-from-v31)
- [修改分区（添加/删除分区和修改分区属性）](#modify-partition)
- [修改分桶方法和桶数（从 v3.2 版本开始）](#modify-the-bucketing-method-and-number-of-buckets-from-v32)
- [修改列（添加/删除列和更改列的顺序）](#modify-columns-adddelete-columns-change-the-order-of-columns)
- [创建/删除 Rollup 索引](#modify-rollup-index)
- [修改 Bitmap 索引](#modify-bitmap-indexes)
- [修改表属性](#modify-table-properties)
- [原子交换](#swap)
- [手动数据版本压缩（从 3.1 版本开始）](#manual-compaction-from-31)

> **注意**
> 此操作需要对目标表具有 ALTER 权限。

## 语法

```SQL
ALTER TABLE [<db_name>.]<tbl_name>
alter_clause1[, alter_clause2, ...]
```

`alter_clause` 可以包含以下操作：重命名、注释、分区、桶、列、Rollup 索引、Bitmap 索引、表属性、交换和压缩。

- rename: 重命名表、Rollup 索引或分区。**注意，列名无法修改。**
- comment: 修改表注释（从 **v3.1** 版本开始支持）。
- partition: 修改分区属性、删除分区或添加分区。
- bucket: 修改分桶方法和桶数。
- column: 添加、删除或重新排序列，或修改列类型。
- Rollup 索引：创建或删除 Rollup 索引。
- Bitmap 索引：修改索引（只能修改 Bitmap 索引）。
- swap: 两个表的原子交换。
- compaction: 执行手动压缩以合并已加载数据的版本（从 **v3.1** 版本开始支持）。

:::note

- 不能在一条 ALTER TABLE 语句中同时对分区、列和 Rollup 索引进行操作。
- 对桶、列和 Rollup 索引的操作是异步的。任务提交后会立即返回成功消息。您可以运行 [SHOW ALTER TABLE](../data-manipulation/SHOW_ALTER.md) 命令来检查进度，并运行 [CANCEL ALTER TABLE](../data-definition/CANCEL_ALTER_TABLE.md) 命令来取消操作。
- 重命名、注释、分区、Bitmap 索引和交换等操作是同步的，命令返回表示执行已完成。
:::

### 重命名

重命名支持修改表名、Rollup 索引和分区名。

#### 重命名表

```sql
ALTER TABLE <tbl_name> RENAME <new_tbl_name>
```

#### 重命名 Rollup 索引

```sql
ALTER TABLE [<db_name>.]<tbl_name>
RENAME ROLLUP <old_rollup_name> <new_rollup_name>
```

#### 重命名分区

```sql
ALTER TABLE [<db_name>.]<tbl_name>
RENAME PARTITION <old_partition_name> <new_partition_name>
```

### 修改表注释（从 v3.1 版本开始）

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

1. Partition_desc 支持以下两种表达式：

   ```plain
   VALUES LESS THAN [MAXVALUE|("value1", ...)]
   VALUES ("value1", ...), ("value1", ...)
   ```

2. 分区是左闭右开区间。如果用户只指定右边界，系统将自动确定左边界。
3. 如果不指定桶方式，则自动使用表内建的桶方式。
4. 如果指定了桶模式，则只能修改桶数，不能修改桶模式或桶列。
5. 用户可以在 `["key"="value"]` 中设置分区的一些属性。详细信息请参见 [CREATE TABLE](CREATE_TABLE.md)。

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

1. 分区表至少应保留一个分区。
2. 执行 DROP PARTITION 一段时间后，可以通过 RECOVER 语句恢复已删除的分区。详细信息请参见 RECOVER 语句。
3. 如果执行 DROP PARTITION FORCE，分区将直接删除，且在不检查分区上是否有未完成活动的情况下无法恢复。因此，通常不建议执行此操作。

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
  - storage_cooldown_ttl 或 storage_cooldown_time
  - replication_num

- 对于只有一个分区的表，分区名与表名相同。如果表分为多个分区，可以使用 `(*)` 来修改所有分区的属性，这更方便。

- 执行 `SHOW PARTITIONS FROM <tbl_name>` 可以查看修改后的分区属性。

### 修改分桶方法和桶数（从 v3.2 版本开始）

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

示例：

例如，原始表是一个 Duplicate Key 表，使用哈希分桶，桶数由 StarRocks 自动设置。

```SQL
CREATE TABLE IF NOT EXISTS details (
    event_time DATETIME NOT NULL COMMENT "事件发生的日期时间",
    event_type INT NOT NULL COMMENT "事件类型",
    user_id INT COMMENT "用户 ID",
    device_code INT COMMENT "设备代码",
    channel INT COMMENT "渠道"
)
DUPLICATE KEY(event_time, event_type)
PARTITION BY date_trunc('day', event_time)
DISTRIBUTED BY HASH(user_id);

-- 插入几天的数据
INSERT INTO details (event_time, event_type, user_id, device_code, channel) VALUES
-- 11月26日的数据
('2023-11-26 08:00:00', 1, 101, 12345, 2),
('2023-11-26 09:15:00', 2, 102, 54321, 3),
('2023-11-26 10:30:00', 1, 103, 98765, 1),
-- 11月27日的数据
('2023-11-27 08:30:00', 1, 104, 11111, 2),
('2023-11-27 09:45:00', 2, 105, 22222, 3),
('2023-11-27 11:00:00', 1, 106, 33333, 1),
-- 11月28日的数据
('2023-11-28 08:00:00', 1, 107, 44444, 2),
('2023-11-28 09:15:00', 2, 108, 55555, 3),
('2023-11-28 10:30:00', 1, 109, 66666, 1);
```

#### 仅修改分桶方法

> **注意**
- 修改将应用于表中的所有分区，不能仅应用于特定分区。
- 虽然只需要修改分桶方法，但命令中仍需使用 `BUCKETS <num>` 指定桶数。如果未指定 `BUCKETS <num>`，则表示桶数由 StarRocks 自动确定。

- 将分桶方法从哈希分桶修改为随机分桶，桶数仍由 StarRocks 自动设置。

  ```SQL
  ALTER TABLE details DISTRIBUTED BY RANDOM;
  ```

- 将哈希分桶的键从 `event_time, event_type` 修改为 `user_id, event_time`，桶数仍由 StarRocks 自动设置。

  ```SQL
  ALTER TABLE details DISTRIBUTED BY HASH(user_id, event_time);
  ```

#### 仅修改桶数

> **注意**
> 虽然只需要修改桶数，但命令中仍需指定分桶方法，例如 `HASH(user_id)`。

- 将所有分区的桶数从 StarRocks 自动设置修改为 10。

  ```SQL
  ALTER TABLE details DISTRIBUTED BY HASH(user_id) BUCKETS 10;
  ```

- 将指定分区的桶数从 StarRocks 自动设置修改为 15。

  ```SQL
  ALTER TABLE details PARTITIONS (p20231127, p20231128) DISTRIBUTED BY HASH(user_id) BUCKETS 15;
  ```

    > **注意**
    > 可以通过执行 `SHOW PARTITIONS FROM <table_name>;` 来查看分区名称。

#### 同时修改分桶方法和桶数

> **注意**
```
```SQL
ALTER TABLE details DISTRIBUTED BY RANDOM BUCKETS 10;
```

```SQL
ALTER TABLE details DISTRIBUTED BY HASH(user_id, event_time) BUCKETS 10;
```

### 修改列（添加/删除列、更改列顺序）

#### 添加一列到指定索引的指定位置

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ADD COLUMN column_name column_type [KEY | agg_type] [DEFAULT "default_value"]
[AFTER column_name|FIRST]
[TO rollup_index_name]
[PROPERTIES ("key"="value", ...)]
```

注意：

1. 如果向聚合表添加值列，需要指定 agg_type。
2. 如果向非聚合表（如 Duplicate Key 表）添加键列，需要指定 KEY 关键字。
3. 不能将基础索引中已存在的列添加到 rollup 索引中。（如有需要，可以重新创建 rollup 索引。）

#### 向指定索引添加多列

语法：

- 添加多列

  ```sql
  ALTER TABLE [<db_name>.]<tbl_name>
  ADD COLUMN (column_name1 column_type [KEY | agg_type] DEFAULT "default_value", ...)
  [TO rollup_index_name]
  [PROPERTIES ("key"="value", ...)]
  ```

- 添加多列并使用 AFTER 指定添加列的位置

  ```sql
  ALTER TABLE [<db_name>.]<tbl_name>
  ADD COLUMN column_name1 column_type [KEY | agg_type] DEFAULT "default_value" AFTER column_name,
  ADD COLUMN column_name2 column_type [KEY | agg_type] DEFAULT "default_value" AFTER column_name
  [, ...]
  [TO rollup_index_name]
  [PROPERTIES ("key"="value", ...)]
  ```

注意：

1. 如果向聚合表添加值列，需要指定 agg_type。

2. 如果向非聚合表添加键列，需要指定 KEY 关键字。

3. 不能将基础索引中已存在的列添加到 rollup 索引中。（如有需要，可以创建另一个 rollup 索引。）

#### 添加生成列（从 v3.1 开始）

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ADD COLUMN col_name data_type [NULL] AS generation_expr [COMMENT 'string']
```

您可以添加一个生成列并指定其表达式。[生成列](../generated_columns.md)可以用于预计算和存储表达式的结果，这显著加速了包含相同复杂表达式的查询。从 v3.1 开始，StarRocks 支持生成列。

#### 从指定索引中删除列

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP COLUMN column_name
[FROM rollup_index_name];
```

注意：

1. 不能删除分区列。
2. 如果从基础索引中删除列，且该列包含在 rollup 索引中，也会被删除。

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

1. 如果修改聚合模型中的值列，需要指定 agg_type。
2. 如果在非聚合模型中修改键列，需要指定 KEY 关键字。
3. 只能修改列的类型，列的其他属性保持当前状态。（即其他属性需要根据原始属性在语句中明确写出，参见[列](#column)部分的示例 8）。
4. 分区列不能修改。
5. 目前支持以下类型的转换（用户需保证精度损失）：

   - 将 TINYINT/SMALLINT/INT/BIGINT 转换为 TINYINT/SMALLINT/INT/BIGINT/DOUBLE。
   - 将 TINYINT/SMALLINT/INT/BIGINT/LARGEINT/FLOAT/DOUBLE/DECIMAL 转换为 VARCHAR。VARCHAR 支持修改最大长度。
   - 将 VARCHAR 转换为 TINYINT/SMALLINT/INT/BIGINT/LARGEINT/FLOAT/DOUBLE。
   - 将 VARCHAR 转换为 DATE（目前支持六种格式：“%Y-%m-%d”、“%y-%m-%d”、“%Y%m%d”、“%y%m%d”、“%Y/%m/%d”、“%y/%m/%d”）
   - 将 DATETIME 转换为 DATE（仅保留年月日信息，例如：`2019-12-09 21:47:05` <--> `2019-12-09`）
   - 将 DATE 转换为 DATETIME（将时、分、秒设置为零，例如：`2019-12-09` <--> `2019-12-09 00:00:00`）
   - 将 FLOAT 转换为 DOUBLE
   - 将 INT 转换为 DATE（如果 INT 数据转换失败，原始数据保持不变）

6. 不支持从 NULL 转换为 NOT NULL。

#### 对指定索引的列重新排序

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ORDER BY (column_name1, column_name2, ...)
[FROM rollup_index_name]
[PROPERTIES ("key"="value", ...)]
```

注意：

- 索引中的所有列都必须写入。
- 值列应在键列之后列出。

#### 修改主键表中排序键的列

<!--Supported Versions-->


语法：

```SQL
ALTER TABLE [<db_name>.]<table_name>
[ order_desc ]

order_desc ::=
    ORDER BY <column_name> [, <column_name> ...]
```

示例：

例如，原表是一个主键表，排序键和主键是耦合的，即 `dt, order_id`。

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

将排序键与主键解耦，并将排序键修改为 `dt, revenue, state`。

```SQL
ALTER TABLE orders ORDER BY (dt, revenue, state);
```

### 修改 rollup 索引

#### 创建 rollup 索引

语法：

```SQL
ALTER TABLE [<db_name>.]<tbl_name> 
ADD ROLLUP rollup_name (column_name1, column_name2, ...)
[FROM from_index_name]
[PROPERTIES ("key"="value", ...)]
```

PROPERTIES：支持设置超时时间，默认超时时间为一天。

示例：

```SQL
ALTER TABLE [<db_name>.]<tbl_name> 
ADD ROLLUP r1(col1, col2) FROM r0;
```

#### 批量创建 rollup 索引

语法：

```SQL
ALTER TABLE [<db_name>.]<tbl_name>
ADD ROLLUP [rollup_name (column_name1, column_name2, ...)
[FROM from_index_name]
[PROPERTIES ("key"="value", ...)],...];
```

示例：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ADD ROLLUP r1(col1, col2) FROM r0, r2(col3, col4) FROM r0;
```

注意：

1. 如果未指定 from_index_name，则默认从基础索引创建。
2. rollup 表中的列必须是 from_index 中的现有列。
3. 在 PROPERTIES 中，用户可以指定存储格式。详见 CREATE TABLE。

#### 删除 rollup 索引

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP ROLLUP rollup_name [PROPERTIES ("key"="value", ...)];
```

示例：

```sql
ALTER TABLE [<db_name>.]<tbl_name> DROP ROLLUP r1;
```

#### 批量删除 rollup 索引

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP ROLLUP [rollup_name [PROPERTIES ("key"="value", ...)],...];
```

示例：

```sql
ALTER TABLE [<db_name>.]<tbl_name> DROP ROLLUP r1, r2;
```

注意：不能删除基础索引。

### 修改位图索引

位图索引支持以下修改：

#### 创建位图索引

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ADD INDEX index_name (column [, ...]) [USING BITMAP] [COMMENT 'balabala'];
```

注意：

```plain
1. 位图索引仅支持当前版本。
2. 位图索引仅在单列中创建。
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
SET ("key" = "value", ...)
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
- `bucket_size`（自 3.2 起支持）

注意：
您还可以通过合并到上述列操作中来修改属性。详见[以下示例](#examples)。

### 交换

Swap 支持两个表的原子交换。

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
SWAP WITH <tbl_name>;
```

### 手动压缩（自 3.1 起）

StarRocks 使用压缩机制合并不同版本的加载数据。该功能可以将小文件合并成大文件，有效提高查询性能。

```
```sql
-- 在整个表上执行压缩。
ALTER TABLE <tbl_name> COMPACT

-- 在单个分区上执行压缩。
ALTER TABLE <tbl_name> COMPACT <partition_name>

-- 在多个分区上执行压缩。
ALTER TABLE <tbl_name> COMPACT (<partition1_name>[,<partition2_name>,...])

-- 执行累积压缩。
ALTER TABLE <tbl_name> CUMULATIVE COMPACT (<partition1_name>[,<partition2_name>,...])

-- 执行基础压缩。
ALTER TABLE <tbl_name> BASE COMPACT (<partition1_name>[,<partition2_name>,...])
```

`information_schema`数据库中的`be_compactions`表记录了压缩结果。您可以执行`SELECT * FROM information_schema.be_compactions;`来查询压缩后的数据版本。

## 示例

### 表

1. 修改表的默认副本数，用作新增分区的默认副本数。

   ```sql
   ALTER TABLE example_db.my_table
   SET ("default.replication_num" = "2");
   ```

2. 修改单分区表的实际副本数。

   ```sql
   ALTER TABLE example_db.my_table
   SET ("replication_num" = "3");
   ```

3. 修改副本之间的数据写入和复制模式。

   ```sql
   ALTER TABLE example_db.my_table
   SET ("replicated_storage" = "false");
   ```

   此示例将副本之间的数据写入和复制模式设置为“无主复制”，即数据同时写入多个副本，不区分主副本和从副本。有关更多信息，请参阅[CREATE TABLE](CREATE_TABLE.md)中的`replicated_storage`参数。

### 分区

1. 添加分区并使用默认的分桶模式。现有分区是[MIN, 2013-01-01)。添加的分区是[2013-01-01, 2014-01-01)。

   ```sql
   ALTER TABLE example_db.my_table
   ADD PARTITION p1 VALUES LESS THAN ("2014-01-01");
   ```

2. 添加分区并使用新的桶数。

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

4. 修改分区的副本数。

   ```sql
   ALTER TABLE example_db.my_table
   MODIFY PARTITION p1 SET("replication_num"="1");
   ```

5. 批量修改指定分区的副本数。

   ```sql
   ALTER TABLE example_db.my_table
   MODIFY PARTITION (p1, p2, p4) SET("replication_num"="1");
   ```

6. 批量修改所有分区的存储介质。

   ```sql
   ALTER TABLE example_db.my_table
   MODIFY PARTITION (*) SET("storage_medium"="HDD");
   ```

7. 删除分区。

   ```sql
   ALTER TABLE example_db.my_table
   DROP PARTITION p1;
   ```

8. 添加具有上下界的分区。

   ```sql
   ALTER TABLE example_db.my_table
   ADD PARTITION p1 VALUES [("2014-01-01"), ("2014-02-01"));
   ```

### Rollup 索引

1. 基于基础索引(k1,k2,k3,v1,v2)创建Rollup索引`example_rollup_index`。使用列式存储。

   ```sql
   ALTER TABLE example_db.my_table
   ADD ROLLUP example_rollup_index(k1, k3, v1, v2)
   PROPERTIES("storage_type"="column");
   ```

2. 基于`example_rollup_index(k1,k3,v1,v2)`创建索引`example_rollup_index2`。

   ```sql
   ALTER TABLE example_db.my_table
   ADD ROLLUP example_rollup_index2 (k1, v1)
   FROM example_rollup_index;
   ```

3. 基于基础索引(k1, k2, k3, v1)创建索引`example_rollup_index3`。设置Rollup超时时间为一小时。

   ```sql
   ALTER TABLE example_db.my_table
   ADD ROLLUP example_rollup_index3(k1, k3, v1)
   PROPERTIES("storage_type"="column", "timeout" = "3600");
   ```

4. 删除索引`example_rollup_index2`。

   ```sql
   ALTER TABLE example_db.my_table
   DROP ROLLUP example_rollup_index2;
   ```

### 列

1. 在`example_rollup_index`的`col1`列之后添加键列`new_col`（非聚合列）。

   ```sql
   ALTER TABLE example_db.my_table
   ADD COLUMN new_col INT KEY DEFAULT "0" AFTER col1
   TO example_rollup_index;
   ```

2. 在`example_rollup_index`的`col1`列之后添加值列`new_col`（非聚合列）。

   ```sql
   ALTER TABLE example_db.my_table
   ADD COLUMN new_col INT DEFAULT "0" AFTER col1
   TO example_rollup_index;
   ```

3. 在`example_rollup_index`的`col1`列之后添加键列`new_col`（聚合列）。

   ```sql
   ALTER TABLE example_db.my_table
   ADD COLUMN new_col INT DEFAULT "0" AFTER col1
   TO example_rollup_index;
   ```

4. 在`example_rollup_index`的`col1`列之后添加值列`new_col SUM`（聚合列）。

   ```sql
   ALTER TABLE example_db.my_table
   ADD COLUMN new_col INT SUM DEFAULT "0" AFTER col1
   TO example_rollup_index;
   ```

5. 将多个列添加到`example_rollup_index`（聚合）。

   ```sql
   ALTER TABLE example_db.my_table
   ADD COLUMN (col1 INT DEFAULT "1", col2 FLOAT SUM DEFAULT "2.3")
   TO example_rollup_index;
   ```

6. 将多个列添加到`example_rollup_index`（聚合），并使用`AFTER`指定添加列的位置。

   ```sql
   ALTER TABLE example_db.my_table
   ADD COLUMN col1 INT DEFAULT "1" AFTER `k1`,
   ADD COLUMN col2 FLOAT SUM AFTER `v2`,
   TO example_rollup_index;
   ```

7. 从`example_rollup_index`中删除列。

   ```sql
   ALTER TABLE example_db.my_table
   DROP COLUMN col2
   FROM example_rollup_index;
   ```

8. 修改基础索引的`col1`列类型为`BIGINT`，并放在`col2`后面。

   ```sql
   ALTER TABLE example_db.my_table
   MODIFY COLUMN col1 BIGINT DEFAULT "1" AFTER col2;
   ```

9. 修改基础索引的`val1`列的最大长度为64。原来的长度是32。

   ```sql
   ALTER TABLE example_db.my_table
   MODIFY COLUMN val1 VARCHAR(64) REPLACE DEFAULT "abc";
   ```

10. 对`example_rollup_index`中的列重新排序。原始列顺序是k1, k2, k3, v1, v2。

    ```sql
    ALTER TABLE example_db.my_table
    ORDER BY (k3,k1,k2,v2,v1)
    FROM example_rollup_index;
    ```

11. 同时执行两项操作（`ADD COLUMN`和`ORDER BY`）。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN v2 INT MAX DEFAULT "0" AFTER k2 TO example_rollup_index,
    ORDER BY (k3,k1,k2,v2,v1) FROM example_rollup_index;
    ```

12. 更改表的布隆过滤器列。

    ```sql
    ALTER TABLE example_db.my_table
    SET ("bloom_filter_columns"="k1,k2,k3");
    ```

    此操作也可以合并到上述列操作中（注意多个子句的语法略有不同）。

    ```sql
    ALTER TABLE example_db.my_table
    DROP COLUMN col2
    PROPERTIES ("bloom_filter_columns"="k1,k2,k3");
    ```

### 表属性

1. 更改表的`Colocate`属性。

   ```sql
   ALTER TABLE example_db.my_table
   SET ("colocate_with" = "t1");
   ```

2. 更改表的动态分区属性。

   ```sql
   ALTER TABLE example_db.my_table
   SET ("dynamic_partition.enable" = "false");
   ```

   如果需要为未配置动态分区属性的表添加动态分区属性，则需要指定所有动态分区属性。

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

1. 将`table1`重命名为`table2`。

   ```sql
   ALTER TABLE table1 RENAME table2;
   ```

2. 将`example_table`的Rollup索引`rollup1`重命名为`rollup2`。

   ```sql
   ALTER TABLE example_table RENAME ROLLUP rollup1 rollup2;
   ```

3. 将`example_table`的分区`p1`重命名为`p2`。

   ```sql
   ALTER TABLE example_table RENAME PARTITION p1 p2;
   ```

### 位图索引

1. 为`table1`中的列`siteid`创建位图索引。

   ```sql
   ALTER TABLE table1
   ADD INDEX index_1 (siteid) [USING BITMAP] COMMENT 'balabala';
   ```

2. 删除`table1`中`siteid`列的位图索引。

   ```sql
   ALTER TABLE table1
   DROP INDEX index_1;
   ```

### 交换

`table1`和`table2`之间的原子交换。

```sql
ALTER TABLE table1 SWAP WITH table2
```

### 手动压缩

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
```
# ALTER TABLE

## 描述

修改现有表，包括：

- [重命名表、分区、索引](#重命名)
- [修改表注释](#从v31开始修改表注释)
- [修改分区（添加/删除分区和修改分区属性）](#修改分区)
- [修改分桶方法和桶数](#从v32开始修改分桶方法和桶数)
- [修改列（添加/删除列和更改列顺序）](#修改列添加删除列更改列顺序)
- [创建/删除滚动索引](#修改滚动索引)
- [修改位图索引](#修改位图索引)
- [修改表属性](#修改表属性)
- [原子交换](#交换)
- [手动数据版本压缩](#从v31开始手动压缩)

> **注意**
> 此操作需要对目标表具有ALTER权限。

## 语法

```SQL
ALTER TABLE [<db_name>.]<tbl_name>
alter_clause1[, alter_clause2, ...]
```

`alter_clause` 可以包含以下操作：重命名、注释、分区、桶、列、滚动索引、位图索引、表属性、交换和压缩。

- 重命名：重命名表、滚动索引或分区。**注意，无法修改列名。**
- 注释：修改表注释（从**v3.1**开始支持）。
- 分区：修改分区属性、删除分区或添加分区。
- 桶：修改分桶方法和桶数。
- 列：添加、删除或重新排序列，或修改列类型。
- 滚动索引：创建或删除滚动索引。
- 位图索引：修改索引（只能修改位图索引）。
- 交换：两个表之间的原子交换。
- 压缩：执行手动压缩以合并加载的数据的版本（从**v3.1**开始支持）。

:::note

- 无法在一个ALTER TABLE语句中执行分区、列和滚动索引的操作。
- 桶、列和滚动索引的操作是异步操作。任务提交后立即返回成功消息。您可以运行[SHOW ALTER TABLE](../data-manipulation/SHOW_ALTER.md)命令来检查进度，并运行[CANCEL ALTER TABLE](../data-definition/CANCEL_ALTER_TABLE.md)命令来取消操作。
- 重命名、注释、分区、位图索引和交换的操作是同步操作，命令返回表示执行完成。
:::

### 重命名

重命名支持修改表名、滚动索引名和分区名。

#### 重命名表

```sql
ALTER TABLE <tbl_name> RENAME <new_tbl_name>
```

#### 重命名滚动索引

```sql
ALTER TABLE [<db_name>.]<tbl_name>
RENAME ROLLUP <old_rollup_name> <new_rollup_name>
```

#### 重命名分区

```sql
ALTER TABLE [<db_name>.]<tbl_name>
RENAME PARTITION <old_partition_name> <new_partition_name>
```

### 从v3.1开始修改表注释

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

1. partition_desc支持以下两种表达式：

   ```plain
   VALUES LESS THAN [MAXVALUE|("value1", ...)]
   VALUES ("value1", ...), ("value1", ...)
   ```

2. 分区是左闭右开区间。如果用户只指定了右边界，则系统会自动确定左边界。
3. 如果未指定桶模式，则自动使用内置表使用的桶方法。
4. 如果指定了桶模式，则只能修改桶数，不能修改桶模式或桶列。
5. 用户可以在`["key"="value"]`中设置分区的一些属性。详细信息请参见[CREATE TABLE](CREATE_TABLE.md)。

#### 删除分区

语法：

```sql
-- 2.0之前
ALTER TABLE [<db_name>.]<tbl_name>
DROP PARTITION [IF EXISTS | FORCE] <partition_name>
-- 2.0或之后
ALTER TABLE [<db_name>.]<tbl_name>
DROP PARTITION [IF EXISTS] <partition_name> [FORCE]
```

注意：

1. 对于分区表，至少保留一个分区。
2. 执行DROP PARTITION一段时间后，可以通过RECOVER语句恢复已删除的分区。有关详细信息，请参见RECOVER语句。
3. 如果执行了DROP PARTITION FORCE，则分区将被直接删除，而无需检查分区上是否有任何未完成的活动。因此，通常不建议执行此操作。

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

- 对于只有一个分区的表，分区名称与表名称相同。如果将表分成多个分区，可以使用`(*)`来修改所有分区的属性，这更方便。

- 执行`SHOW PARTITIONS FROM <tbl_name>`以查看修改后的分区属性。

### 从v3.2开始修改分桶方法和桶数

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

示例：

例如，原始表是一个使用哈希分桶的重复键表，桶数由StarRocks自动设置。

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

-- 插入几天的数据
INSERT INTO details (event_time, event_type, user_id, device_code, channel) VALUES
-- 11月26日的数据
('2023-11-26 08:00:00', 1, 101, 12345, 2),
('2023-11-26 09:15:00', 2, 102, 54321, 3),
('2023-11-26 10:30:00', 1, 103, 98765, 1),
-- 11月27日的数据
('2023-11-27 08:30:00', 1, 104, 11111, 2),
('2023-11-27 09:45:00', 2, 105, 22222, 3),
('2023-11-27 11:00:00', 1, 106, 33333, 1),
-- 11月28日的数据
('2023-11-28 08:00:00', 1, 107, 44444, 2),
('2023-11-28 09:15:00', 2, 108, 55555, 3),
('2023-11-28 10:30:00', 1, 109, 66666, 1);
```

#### 仅修改分桶方法

> **注意**
- 修改适用于表中的所有分区，无法仅应用于特定分区。
- 尽管只需要修改分桶方法，但在命令中仍然需要指定桶数，使用`BUCKETS <num>`。如果未指定`BUCKETS <num>`，则表示桶数由StarRocks自动确定。

- 将分桶方法从哈希分桶修改为随机分桶，桶数仍由StarRocks自动设置。

  ```SQL
  ALTER TABLE details DISTRIBUTED BY RANDOM;
  ```

- 将哈希分桶的键修改为`user_id, event_time`，桶数仍由StarRocks自动设置。

  ```SQL
  ALTER TABLE details DISTRIBUTED BY HASH(user_id, event_time);
  ```

#### 仅修改桶数

> **注意**
> 尽管只需要修改桶数，但仍然需要在命令中指定分桶方法，例如`HASH(user_id)`。

- 将所有分区的桶数修改为10，桶数由StarRocks自动设置。

  ```SQL
  ALTER TABLE details DISTRIBUTED BY HASH(user_id) BUCKETS 10;
  ```

- 将指定分区的桶数修改为15，桶数由StarRocks自动设置。

  ```SQL
  ALTER TABLE details PARTITIONS (p20231127, p20231128) DISTRIBUTED BY HASH(user_id) BUCKETS 15 ;
  ```

    > **注意**
    > 可以通过执行`SHOW PARTITIONS FROM <table_name>;`来查看分区名称。

#### 同时修改分桶方法和桶数

> **注意**
> 修改适用于表中的所有分区，无法仅应用于特定分区。

- 将分桶方法从哈希分桶修改为随机分桶，并将桶数修改为10，桶数由StarRocks自动设置。

  ```SQL
  ALTER TABLE details DISTRIBUTED BY RANDOM BUCKETS 10;
  ```

- 将哈希分桶的键修改为`user_id, event_time`，并将桶数修改为10，桶数由StarRocks自动设置。

  ```SQL
  ALTER TABLE details DISTRIBUTED BY HASH(user_id, event_time) BUCKETS 10;
  ```

### 修改列（添加/删除列，更改列顺序）

#### 将列添加到指定索引的指定位置

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ADD COLUMN column_name column_type [KEY | agg_type] [DEFAULT "default_value"]
[AFTER column_name|FIRST]
[TO rollup_index_name]
[PROPERTIES ("key"="value", ...)]
```

注意：

1. 如果向聚合表添加值列，则需要指定agg_type。
2. 如果向非聚合表（例如重复键表）添加键列，则需要指定KEY关键字。
3. 无法将已存在于基本索引中的列添加到滚动索引中（如果需要，可以重新创建滚动索引）。

#### 向指定索引添加多列

语法：

- 添加多列

  ```sql
  ALTER TABLE [<db_name>.]<tbl_name>
  ADD COLUMN (column_name1 column_type [KEY | agg_type] DEFAULT "default_value", ...)
  [TO rollup_index_name]
  [PROPERTIES ("key"="value", ...)]
  ```

- 添加多列并使用AFTER指定添加列的位置

  ```sql
  ALTER TABLE [<db_name>.]<tbl_name>
  ADD COLUMN column_name1 column_type [KEY | agg_type] DEFAULT "default_value" AFTER column_name,
  ADD COLUMN column_name2 column_type [KEY | agg_type] DEFAULT "default_value" AFTER column_name
  [, ...]
  [TO rollup_index_name]
  [PROPERTIES ("key"="value", ...)]
  ```

注意：

1. 如果向聚合表添加值列，则需要指定agg_type。
2. 如果向非聚合表添加键列（例如重复键表），则需要指定KEY关键字。
3. 无法将已存在于基本索引中的列添加到滚动索引中（如果需要，可以创建另一个滚动索引）。

#### 添加生成列（从v3.1开始）

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ADD COLUMN col_name data_type [NULL] AS generation_expr [COMMENT 'string']
```

您可以添加生成列并指定其表达式。[生成列](../generated_columns.md)可用于预计算和存储表达式的结果，从而显著加快具有相同复杂表达式的查询。从v3.1开始，StarRocks支持生成列。

#### 从指定索引中删除列

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP COLUMN column_name
[FROM rollup_index_name];
```

注意：

1. 无法删除分区列。
2. 如果从基本索引中删除了列，并且该列包含在滚动索引中，则该列也将被删除。

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

1. 如果要修改聚合模型中的值列，需要指定agg_type。
2. 如果要修改非聚合模型中的键列，需要指定KEY关键字。
3. 仅可以修改列的类型，列的其他属性保持不变（即其他属性需要根据原始属性在语句中显式编写，参见[column](#column)部分中的示例8）。
4. 无法修改分区列。
5. 目前支持以下类型的转换（用户保证精度损失）。

   - 将TINYINT/SMALLINT/INT/BIGINT转换为TINYINT/SMALLINT/INT/BIGINT/DOUBLE。
   - 将TINTINT/SMALLINT/INT/BIGINT/LARGEINT/FLOAT/DOUBLE/DECIMAL转换为VARCHAR。VARCHAR支持修改最大长度。
   - 将VARCHAR转换为TINTINT/SMALLINT/INT/BIGINT/LARGEINT/FLOAT/DOUBLE。
   - 将VARCHAR转换为DATE（当前支持六种格式：“%Y-%m-%d”，“%y-%m-%d”，“%Y%m%d”，“%y%m%d”，“%Y/%m/%d”，“%y/%m/%d”）
   - 将DATETIME转换为DATE（仅保留年-月-日信息，即`2019-12-09 21:47:05` `<-->` `2019-12-09`）
   - 将DATE转换为DATETIME（将小时、分钟、秒设置为零，例如：`2019-12-09` `<-->` `2019-12-09 00:00:00`）
   - 将FLOAT转换为DOUBLE
   - 将INT转换为DATE（如果INT数据无法转换，则保持原始数据不变）

6. 不支持从NULL转换为NOT NULL。

#### 重新排序指定索引的列

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ORDER BY (column_name1, column_name2, ...)
[FROM rollup_index_name]
[PROPERTIES ("key"="value", ...)]
```

注意：

- 必须列出索引中的所有列。
- 值列在键列之后列出。

#### 修改主键表中排序键的列

<!--Supported Versions-->


语法：

```SQL
ALTER TABLE [<db_name>.]<table_name>
[ order_desc ]

order_desc ::=
    ORDER BY <column_name> [, <column_name> ...]
```

示例：

例如，原始表是一个主键表，其中排序键和主键是耦合的，即`dt, order_id`。

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

将排序键从主键中解耦，并将排序键修改为`dt, revenue, state`。

```SQL
ALTER TABLE orders ORDER BY (dt, revenue, state);
```

### 修改滚动索引

#### 创建滚动索引

语法：

```SQL
ALTER TABLE [<db_name>.]<tbl_name> 
ADD ROLLUP rollup_name (column_name1, column_name2, ...)
[FROM from_index_name]
[PROPERTIES ("key"="value", ...)]
```

PROPERTIES：支持设置超时时间，默认超时时间为一天。

示例：

```SQL
ALTER TABLE [<db_name>.]<tbl_name> 
ADD ROLLUP r1(col1,col2) from r0;
```

#### 批量创建滚动索引

语法：

```SQL
ALTER TABLE [<db_name>.]<tbl_name>
ADD ROLLUP [rollup_name (column_name1, column_name2, ...)
[FROM from_index_name]
[PROPERTIES ("key"="value", ...)],...];
```

示例：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ADD ROLLUP r1(col1,col2) from r0, r2(col3,col4) from r0;
```

注意：

1. 如果未指定from_index_name，则默认从基本索引创建。
2. 滚动表中的列必须是from_index中的现有列。
3. 在properties中，用户可以指定存储格式。有关详细信息，请参见CREATE TABLE。

#### 删除滚动索引

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP ROLLUP rollup_name [PROPERTIES ("key"="value", ...)];
```

示例：

```sql
ALTER TABLE [<db_name>.]<tbl_name> DROP ROLLUP r1;
```

#### 批量删除滚动索引

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP ROLLUP [rollup_name [PROPERTIES ("key"="value", ...)],...];
```

示例：

```sql
ALTER TABLE [<db_name>.]<tbl_name> DROP ROLLUP r1, r2;
```

注意：无法删除基本索引。

### 修改位图索引

位图索引支持以下修改：

#### 创建位图索引

语法：

```sql
 ALTER TABLE [<db_name>.]<tbl_name>
ADD INDEX index_name (column [, ...],) [USING BITMAP] [COMMENT 'balabala'];
```

注意：

```plain
1. 位图索引仅支持当前版本。
2. 仅在单个列中创建BITMAP索引。
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

目前，StarRocks支持修改以下表属性：

- `replication_num`
- `default.replication_num`
- `storage_cooldown_ttl`
- `storage_cooldown_time`
- 动态分区相关属性
- `enable_persistent_index`
- `bloom_filter_columns`
- `colocate_with`
- `bucket_size`（从3.2开始支持）

注意：
您还可以通过合并到列上的上述操作来修改属性。请参见[以下示例](#示例)。

### 交换

交换支持两个表之间的原子交换。

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
SWAP WITH <tbl_name>;
```

### 手动压缩（从3.1开始）

StarRocks使用压缩机制来合并加载的数据的不同版本。此功能可以将小文件合并为大文件，从而有效提高查询性能。

在v3.1之前，压缩有两种方式：

- 系统自动压缩：在后台的BE级别执行压缩。用户无法指定数据库或表进行压缩。
- 用户可以通过调用HTTP接口来执行压缩。

从v3.1开始，StarRocks为用户提供了SQL接口，可以通过运行SQL命令来手动执行压缩。他们可以选择特定的表或分区进行压缩。这提供了对压缩过程的更大灵活性和控制。

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

-- 执行基本压缩。
ALTER TABLE <tbl_name> BASE COMPACT (<partition1_name>[,<partition2_name>,...])
```

`information_schema`数据库中的`be_compactions`表记录了压缩结果。您可以运行`SELECT * FROM information_schema.be_compactions;`来查询压缩后的数据版本。

## 示例

### 表

1. 修改表的默认副本数，用作新添加分区的默认副本数。

   ```sql
   ALTER TABLE example_db.my_table
   SET ("default.replication_num" = "2");
   ```

2. 修改单分区表的实际副本数。

   ```sql
   ALTER TABLE example_db.my_table
   SET ("replication_num" = "3");
   ```

3. 修改数据写入和副本模式。

   ```sql
   ALTER TABLE example_db.my_table
   SET ("replicated_storage" = "false");
   ```

   此示例将数据写入和副本模式设置为“无领导复制”，这意味着数据同时写入多个副本，而不区分主副本和次副本。有关详细信息，请参见[CREATE TABLE](CREATE_TABLE.md)中的`replicated_storage`参数。

### 分区

1. 添加一个分区，并使用默认的分桶模式。现有的分区是[MIN，2013-01-01)。添加的分区是[2013-01-01，2014-01-01)。

   ```sql
   ALTER TABLE example_db.my_table
   ADD PARTITION p1 VALUES LESS THAN ("2014-01-01");
   ```

2. 添加一个分区，并使用新的桶数。

   ```sql
   ALTER TABLE example_db.my_table
   ADD PARTITION p1 VALUES LESS THAN ("2015-01-01")
   DISTRIBUTED BY HASH(k1);
   ```

3. 添加一个分区，并使用新的副本数。

   ```sql
   ALTER TABLE example_db.my_table
   ADD PARTITION p1 VALUES LESS THAN ("2015-01-01")
   ("replication_num"="1");
   ```

4. 修改分区的副本数。

   ```sql
   ALTER TABLE example_db.my_table
   MODIFY PARTITION p1 SET("replication_num"="1");
   ```

5. 批量修改指定分区的副本数。

   ```sql
   ALTER TABLE example_db.my_table
   MODIFY PARTITION (p1, p2, p4) SET("replication_num"="1");
   ```

6. 批量修改所有分区的存储介质。

   ```sql
   ALTER TABLE example_db.my_table
   MODIFY PARTITION (*) SET("storage_medium"="HDD");
   ```

7. 删除一个分区。

   ```sql
   ALTER TABLE example_db.my_table
   DROP PARTITION p1;
   ```

8. 添加一个具有上下界的分区。

   ```sql
   ALTER TABLE example_db.my_table
   ADD PARTITION p1 VALUES [("2014-01-01"), ("2014-02-01"));
   ```

### 滚动索引

1. 在基本索引（k1，k2，k3，v1，v2）上创建一个名为`example_rollup_index`的滚动索引，使用列存储。

   ```sql
   ALTER TABLE example_db.my_table
   ADD ROLLUP example_rollup_index(k1, k3, v1, v2)
   PROPERTIES("storage_type"="column");
   ```

2. 在`example_rollup_index(k1，k3，v1，v2)`上创建一个名为`example_rollup_index2`的索引。

   ```sql
   ALTER TABLE example_db.my_table
   ADD ROLLUP example_rollup_index2 (k1, v1)
   FROM example_rollup_index;
   ```

3. 在基本索引（k1，k2，k3，v1）上创建一个名为`example_rollup_index3`的索引，将滚动超时时间设置为一小时。

   ```sql
   ALTER TABLE example_db.my_table
   ADD ROLLUP example_rollup_index3(k1, k3, v1)
   PROPERTIES("storage_type"="column", "timeout" = "3600");
   ```

4. 删除索引`example_rollup_index2`。

   ```sql
   ALTER TABLE example_db.my_table
   DROP ROLLUP example_rollup_index2;
   ```

### 列

1. 在`example_rollup_index`的`col1`列之后添加一个键列`new_col`（非聚合列）。

   ```sql
   ALTER TABLE example_db.my_table
   ADD COLUMN new_col INT KEY DEFAULT "0" AFTER col1
   TO example_rollup_index;
   ```

2. 在`example_rollup_index`的`col1`列之后添加一个值列`new_col`（非聚合列）。

   ```sql
   ALTER TABLE example_db.my_table
   ADD COLUMN new_col INT DEFAULT "0" AFTER col1
   TO example_rollup_index;
   ```

3. 在`example_rollup_index`的`col1`列之后添加一个键列`new_col`（聚合列）。

   ```sql
   ALTER TABLE example_db.my_table
   ADD COLUMN new_col INT DEFAULT "0" AFTER col1
   TO example_rollup_index;
   ```

4. 在`example_rollup_index`的`col1`列之后添加一个值列`new_col SUM`（聚合列）。

   ```sql
   ALTER TABLE example_db.my_table
   ADD COLUMN new_col INT SUM DEFAULT "0" AFTER col1
   TO example_rollup_index;
   ```

5. 向`example_rollup_index`（聚合）添加多列。

   ```sql
   ALTER TABLE example_db.my_table
   ADD COLUMN (col1 INT DEFAULT "1", col2 FLOAT SUM DEFAULT "2.3")
   TO example_rollup_index;
   ```

6. 向`example_rollup_index`（聚合）添加多列，并使用AFTER指定添加列的位置。

   ```sql
   ALTER TABLE example_db.my_table
   ADD COLUMN col1 INT DEFAULT "1" AFTER `k1`,
   ADD COLUMN col2 FLOAT SUM AFTER `v2`,
   TO example_rollup_index;
   ```

7. 从`example_rollup_index`中删除一列。

   ```sql
   ALTER TABLE example_db.my_table
   DROP COLUMN col2
   FROM example_rollup_index;
   ```

8. 修改基本索引的col1列的列类型为BIGINT，并将其放在col2之后。

   ```sql
   ALTER TABLE example_db.my_table
   MODIFY COLUMN col1 BIGINT DEFAULT "1" AFTER col2;
   ```

9. 将基本索引的val1列的最大长度修改为64。原始长度为32。

   ```sql
   ALTER TABLE example_db.my_table
   MODIFY COLUMN val1 VARCHAR(64) REPLACE DEFAULT "abc";
   ```

10. 重新排序`example_rollup_index`中的列。原始列顺序为k1，k2，k3，v1，v2。

    ```sql
    ALTER TABLE example_db.my_table
    ORDER BY (k3,k1,k2,v2,v1)
    FROM example_rollup_index;
    ```

11. 同时执行两个操作（ADD COLUMN和ORDER BY）。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN v2 INT MAX DEFAULT "0" AFTER k2 TO example_rollup_index,
    ORDER BY (k3,k1,k2,v2,v1) FROM example_rollup_index;
    ```

12. 修改表的布隆过滤器列。

    ```sql
    ALTER TABLE example_db.my_table
    SET ("bloom_filter_columns"="k1,k2,k3");
    ```

    此操作也可以合并到上述列操作中（注意多个子句的语法略有不同）。

    ```sql
    ALTER TABLE example_db.my_table
    DROP COLUMN col2
    PROPERTIES ("bloom_filter_columns"="k1,k2,k3");
    ```

### 表属性

1. 修改表的Colocate属性。

   ```sql
   ALTER TABLE example_db.my_table
   SET ("colocate_with" = "t1");
   ```

2. 修改表的动态分区属性。

   ```sql
   ALTER TABLE example_db.my_table
   SET ("dynamic_partition.enable" = "false");
   ```

   如果需要向没有配置动态分区属性的表添加动态分区属性，则需要指定所有动态分区属性。

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

1. 将`table1`重命名为`table2`。

   ```sql
   ALTER TABLE table1 RENAME table2;
   ```

2. 将`example_table`的滚动索引`rollup1`重命名为`rollup2`。

   ```sql
   ALTER TABLE example_table RENAME ROLLUP rollup1 rollup2;
   ```

3. 将`example_table`的分区`p1`重命名为`p2`。

   ```sql
   ALTER TABLE example_table RENAME PARTITION p1 p2;
   ```

### 位图索引

1. 为`table1`中的`siteid`列创建一个位图索引。

   ```sql
   ALTER TABLE table1
   ADD INDEX index_1 (siteid) [USING BITMAP] COMMENT 'balabala';
   ```

2. 删除`table1`中`siteid`列的位图索引。

   ```sql
   ALTER TABLE table1
   DROP INDEX index_1;
   ```

### 交换

在`table1`和`table2`之间进行原子交换。

```sql
ALTER TABLE table1 SWAP WITH table2
```

### 手动压缩

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

INSERT INTO compaction_test VALUES
('2023-02-14', 2),('2033-03-01',2);
{'label':'insert_85c95c1b-c878-11ed-90d6-00163e0dcbfc', 'status':'VISIBLE', 'txnId':'5009'}

ALTER TABLE compaction_test COMPACT;

ALTER TABLE compaction_test COMPACT p203303;

ALTER TABLE compaction_test COMPACT (p202302,p203303);

ALTER TABLE compaction_test CUMULATIVE COMPACT (p202302,p203303);

ALTER TABLE compaction_test BASE COMPACT (p202302,p203303);
```

## 参考

- [创建表](./CREATE_TABLE.md)
- [显示创建表](../data-manipulation/SHOW_CREATE_TABLE.md)
- [显示表格](../data-manipulation/SHOW_TABLES.md)
- [显示更改表](../data-manipulation/SHOW_ALTER.md)
- [掉落表](./DROP_TABLE.md)