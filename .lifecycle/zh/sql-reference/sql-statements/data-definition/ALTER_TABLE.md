---
displayed_sidebar: English
---

# 修改现有表

## 描述

对现有表进行修改，包括：

- [重命名表、分区、索引](#rename)
- [修改表的注释](#alter-table-comment-from-v31)
- [修改分区（添加/删除分区以及修改分区属性）](#modify-partition)
- [修改分桶方法和分桶数量](#modify-the-bucketing-method-and-number-of-buckets-from-v32)
- [修改列（添加/删除列以及更改列的顺序）](#modify-columns-adddelete-columns-change-the-order-of-columns)
- [创建/删除rollup索引](#modify-rollup-index)
- [修改位图索引](#modify-bitmap-indexes)
- [修改表属性](#modify-table-properties)
- [原子交换](#swap)
- [手动数据版本压缩](#manual-compaction-from-31)

> **注意**
> 此操作需要对**目标表**拥有**ALTER**权限。

## 语法

```SQL
ALTER TABLE [<db_name>.]<tbl_name>
alter_clause1[, alter_clause2, ...]
```

alter_clause可以进行以下操作：重命名、注释、分区、分桶、列、rollup索引、位图索引、表属性、交换和压缩。

- 重命名：重命名表、rollup索引或分区。**请注意，无法修改列名。**
- 注释：修改表的注释（自**v3.1**版本起支持）。
- 分区：修改分区属性、删除分区或添加分区。
- 分桶：修改分桶方法和桶数量。
- 列：添加、删除或重新排序列，或修改列类型。
- rollup索引：创建或删除rollup索引。
- 位图索引：修改索引（只能修改位图索引）。
- 交换：两个表的原子交换。
- 压缩：手动执行压缩以合并已加载数据的不同版本（从**v3.1版本起**支持）。

:::注意

- 在一条ALTER TABLE语句中不能同时对分区、列和rollup索引进行操作。
- 对分桶、列和rollup索引的操作是异步的。任务提交后会立即返回成功消息。您可以运行[SHOW ALTER TABLE](../data-manipulation/SHOW_ALTER.md)命令检查进度，并使用[CANCEL ALTER TABLE](../data-definition/CANCEL_ALTER_TABLE.md)命令取消操作。
- 重命名、注释、分区、位图索引和交换的操作是同步的，命令返回表示执行已完成。:::

### 重命名

重命名支持修改表名、rollup索引名和分区名。

#### 重命名表

```sql
ALTER TABLE <tbl_name> RENAME <new_tbl_name>
```

#### 重命名rollup索引

```sql
ALTER TABLE [<db_name>.]<tbl_name>
RENAME ROLLUP <old_rollup_name> <new_rollup_name>
```

#### 重命名分区

```sql
ALTER TABLE [<db_name>.]<tbl_name>
RENAME PARTITION <old_partition_name> <new_partition_name>
```

### 修改表注释（从v3.1版开始）

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

1. Partition_desc支持以下两种表达方式：

   ```plain
   VALUES LESS THAN [MAXVALUE|("value1", ...)]
   VALUES ("value1", ...), ("value1", ...)
   ```

2. 分区是左闭右开区间。如果用户只指定右边界，系统将自动确定左边界。
3. 如果未指定分桶方式，则自动采用表内置的分桶方式。
4. 如果指定了分桶方式，则只能修改桶数量，不能修改桶方式或桶列。
5. 用户可以在`["key"="value"]`中设置分区的某些属性。详细信息请参见[CREATE TABLE](CREATE_TABLE.md)。

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
2. 执行DROP PARTITION一段时间后，可以通过RECOVER语句恢复已删除的分区。详细信息请参见RECOVER语句。
3. 如果执行DROP PARTITION FORCE，将直接删除分区，并且不检查分区上是否有未完成的活动，因此无法恢复。通常不建议进行此操作。

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

  - 存储介质
  - storage_cooldown_ttl或storage_cooldown_time
  - 副本数量

- 对于只有一个分区的表，分区名与表名相同。如果表被划分为多个分区，可以使用(*)来方便地修改所有分区的属性。

- 执行SHOW PARTITIONS FROM <tbl_name>后可以查看修改后的分区属性。

### 修改分桶方式和桶数量（从v3.2版本起）

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

例如：

比如原表是一个Duplicate Key表，使用的是哈希分桶，桶数量由StarRocks自动设定。

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
- 修改将应用于表中的所有分区，不能只针对特定分区。
- 尽管只需要修改分桶方式，命令中仍需使用BUCKETS <num>指定桶数量。如果没有指定BUCKETS <num>，则意味着桶数量由StarRocks自动决定。

- 分桶方式从哈希分桶改为随机分桶，桶数量仍由StarRocks自动设定。

  ```SQL
  ALTER TABLE details DISTRIBUTED BY RANDOM;
  ```

- 哈希分桶的键从event_time、event_type修改为user_id、event_time，桶数量仍由StarRocks自动设定。

  ```SQL
  ALTER TABLE details DISTRIBUTED BY HASH(user_id, event_time);
  ```

#### 仅修改桶数量

> **注意**
> 尽管只需要修改桶数量，但命令中仍需指定分桶方式，例如，`HASH(user_id)`。

- 将所有分区的桶数量从StarRocks自动设定的数量修改为10。

  ```SQL
  ALTER TABLE details DISTRIBUTED BY HASH(user_id) BUCKETS 10;
  ```

- 将指定分区的桶数量从StarRocks自动设定的数量修改为15。

  ```SQL
  ALTER TABLE details PARTITIONS (p20231127, p20231128) DISTRIBUTED BY HASH(user_id) BUCKETS 15 ;
  ```

    > **注意**
    > 可以通过执行 `SHOW PARTITIONS FROM \\<table_name\\>;` 查看分区名。

#### 同时修改分桶方式和桶数量

> **注意**
> 修改将应用于表中的所有分区，不能只针对特定分区。

- 将分桶方式从哈希分桶改为随机分桶，并将桶数量从StarRocks自动设定的数量改为10。

  ```SQL
  ALTER TABLE details DISTRIBUTED BY RANDOM BUCKETS 10;
  ```

- 修改哈希分桶的键，并将桶数量从StarRocks自动设定的数量改为10。哈希分桶的键从原来的event_time、event_type修改为user_id、event_time，桶数量修改为10。

  ```SQL
  ALTER TABLE details DISTRIBUTED BY HASH(user_id, event_time) BUCKETS 10;
  ``
  
  ```

### 修改列（添加/删除列、更改列顺序）

#### 在指定索引的指定位置添加一列

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ADD COLUMN column_name column_type [KEY | agg_type] [DEFAULT "default_value"]
[AFTER column_name|FIRST]
[TO rollup_index_name]
[PROPERTIES ("key"="value", ...)]
```

注意：

1. 如果向聚合表中添加值列，需要指定agg_type。
2. 如果向非聚合表（如Duplicate Key表）中添加键列，需要指定KEY关键字。
3. 不能将基础索引中已存在的列添加到rollup索引中。（如有需要，可以重新创建rollup索引。）

#### 向指定索引中添加多列

语法：

- 添加多列

  ```sql
  ALTER TABLE [<db_name>.]<tbl_name>
  ADD COLUMN (column_name1 column_type [KEY | agg_type] DEFAULT "default_value", ...)
  [TO rollup_index_name]
  [PROPERTIES ("key"="value", ...)]
  ```

- 使用AFTER指定添加列的位置来添加多列

  ```sql
  ALTER TABLE [<db_name>.]<tbl_name>
  ADD COLUMN column_name1 column_type [KEY | agg_type] DEFAULT "default_value" AFTER column_name,
  ADD COLUMN column_name2 column_type [KEY | agg_type] DEFAULT "default_value" AFTER column_name
  [, ...]
  [TO rollup_index_name]
  [PROPERTIES ("key"="value", ...)]
  ```

注意：

1. 如果向聚合表中添加值列，需要指定agg_type。

2. 如果向非聚合表中添加键列，需要指定KEY关键字。

3. 不能将基础索引中已存在的列添加到rollup索引中。（如有需要，可以创建另一个rollup索引。）

#### 添加生成列（从v3.1版本起）

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ADD COLUMN col_name data_type [NULL] AS generation_expr [COMMENT 'string']
```

您可以添加生成列并指定其表达式。[生成列](../generated_columns.md)可以用于预计算和存储表达式结果，这显著加快了包含相同复杂表达式的查询速度。从v3.1，StarRocks支持生成列。

#### 从指定索引中删除列

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP COLUMN column_name
[FROM rollup_index_name];
```

注意：

1. 不能删除分区列。
2. 如果从基础索引中删除列，且该列包含在rollup索引中，则该列也将被删除。

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

1. 如果修改聚合模型中的值列，需要指定agg_type。
2. 如果在非聚合模型中修改键列，需要指定KEY关键字。
3. 只能修改列的类型，其他属性保持现状。（即，其他属性需要根据当前的属性在语句中明确指定，参见[列](#column)部分的示例8。）
4. 分区列无法修改。
5. 目前支持以下类型转换（精度损失需由用户自行保证）：

   - 将 TINYINT/SMALLINT/INT/BIGINT 转换为 TINYINT/SMALLINT/INT/BIGINT/DOUBLE。
   - 将 TINYINT/SMALLINT/INT/BIGINT/LARGEINT/FLOAT/DOUBLE/DECIMAL 转换为 VARCHAR。VARCHAR 支持修改最大长度。
   - 将 VARCHAR 转换为 TINYINT/SMALLINT/INT/BIGINT/LARGEINT/FLOAT/DOUBLE。
   - 将 VARCHAR 转换为 DATE（当前支持六种格式：“%Y-%m-%d”、“%y-%m-%d”、“%Y%m%d”、“%y%m%d”、“%Y/%m/%d”、“%y/%m/%d”）。
   - 将 DATETIME 转换为 DATE（只保留年月日信息，即 2019-12-09 21:47:05 <--> 2019-12-09）。
   - 将 DATE 转换为 DATETIME（小时、分钟、秒设为零，例如：2019-12-09 <--> 2019-12-09 00:00:00）。
   - 将 FLOAT 转换为 DOUBLE。
   - 将 INT 转换为 DATE（若 INT 数据无法转换，则保留原始数据）。

6. 不支持从 NULL 转换为 NOT NULL。

#### 对指定索引重排列

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ORDER BY (column_name1, column_name2, ...)
[FROM rollup_index_name]
[PROPERTIES ("key"="value", ...)]
```

注意：

- 必须列出索引中的所有列。
- 值列应位于键列之后。

#### 在主键表中修改排序键列

<!--Supported Versions-->


语法：

```SQL
ALTER TABLE [<db_name>.]<table_name>
[ order_desc ]

order_desc ::=
    ORDER BY <column_name> [, <column_name> ...]
```

示例：

例如，原始表是主键表，其排序键与主键耦合，为 dt 和 order_id。

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

将排序键与主键解耦，并将排序键修改为 dt、revenue、state。

```SQL
ALTER TABLE orders ORDER BY (dt, revenue, state);
```

### 修改 Rollup 索引

#### 创建 Rollup 索引

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

#### 批量创建 Rollup 索引

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

1. 如果未指定 from_index_name，则默认基于基础索引创建。
2. Rollup 表中的列必须是 from_index 中已存在的列。
3. 在 properties 中，用户可以指定存储格式。详细信息请参见 CREATE TABLE。

#### 删除 Rollup 索引

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP ROLLUP rollup_name [PROPERTIES ("key"="value", ...)];
```

示例：

```sql
ALTER TABLE [<db_name>.]<tbl_name> DROP ROLLUP r1;
```

#### 批量删除 Rollup 索引

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

### 修改 Bitmap 索引

Bitmap 索引支持以下修改：

#### 创建 Bitmap 索引

语法：

```sql
 ALTER TABLE [<db_name>.]<tbl_name>
ADD INDEX index_name (column [, ...],) [USING BITMAP] [COMMENT 'balabala'];
```

注意：

```plain
1. Bitmap index is only supported for the current version.
2. A BITMAP index is created only in a single column.
```

#### 删除 Bitmap 索引

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

- 复制数（replication_num）
- 默认复制数（default.replication_num）
- 存储冷却时间（storage_cooldown_ttl）
- 存储冷却时间（storage_cooldown_time）
- 与动态分区相关的属性
- 启用持久性索引（enable_persistent_index）
- 布隆过滤器列（bloom_filter_columns）
- 与其他表共位（colocate_with）
- 桶大小（bucket_size，自 3.2 版本起支持）

注意：您还可以将这些属性修改合并到上述列操作中。请参见[下面的示例](#examples)。

### 交换

Swap 支持两个表之间的原子交换。

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
SWAP WITH <tbl_name>;
```

### 手动压缩（自 3.1 版本起）

StarRocks 使用压缩机制合并不同版本的加载数据。此功能可以将小文件合并成大文件，有效提高查询性能。

在 3.1 版本之前，压缩有两种方式：

- 系统自动压缩：在后台的 BE 级别进行压缩。用户不能指定压缩的数据库或表。
- 用户可以通过调用 HTTP 接口进行压缩。

从 3.1 版本开始，StarRocks 提供 SQL 接口，允许用户通过执行 SQL 命令手动进行压缩。他们可以选择特定的表或分区进行压缩，从而提供更多的灵活性和控制。

语法：

```sql
-- Perform compaction on the entire table.
ALTER TABLE <tbl_name> COMPACT

-- Perform compaction on a single partition.
ALTER TABLE <tbl_name> COMPACT <partition_name>

-- Perform compaction on multiple partitions.
ALTER TABLE <tbl_name> COMPACT (<partition1_name>[,<partition2_name>,...])

-- Perform cumulative compaction.
ALTER TABLE <tbl_name> CUMULATIVE COMPACT (<partition1_name>[,<partition2_name>,...])

-- Perform base compaction.
ALTER TABLE <tbl_name> BASE COMPACT (<partition1_name>[,<partition2_name>,...])
```

information_schema 数据库中的 be_compactions 表记录压缩结果。您可以执行 SELECT * FROM information_schema.be_compactions; 来查询压缩后的数据版本。

## 示例

### 表

1. 修改表的默认副本数量，作为新添加分区的默认副本数量。

   ```sql
   ALTER TABLE example_db.my_table
   SET ("default.replication_num" = "2");
   ```

2. 修改单分区表的实际副本数量。

   ```sql
   ALTER TABLE example_db.my_table
   SET ("replication_num" = "3");
   ```

3. 修改副本间的数据写入和复制模式。

   ```sql
   ALTER TABLE example_db.my_table
   SET ("replicated_storage" = "false");
   ```

   此示例设置了数据写入和副本之间的复制模式为"无主复制"，这意味着数据同时写入多个副本，而不区分主副本和从副本。有关更多信息，请参见[CREATE TABLE](CREATE_TABLE.md)中的`replicated_storage`参数。

### 分区

1. 添加分区并使用默认桶模式。现有分区为 [MIN, 2013-01-01)。添加的分区为 [2013-01-01, 2014-01-01)。

   ```sql
   ALTER TABLE example_db.my_table
   ADD PARTITION p1 VALUES LESS THAN ("2014-01-01");
   ```

2. 添加分区并使用新的桶数量。

   ```sql
   ALTER TABLE example_db.my_table
   ADD PARTITION p1 VALUES LESS THAN ("2015-01-01")
   DISTRIBUTED BY HASH(k1);
   ```

3. 添加分区并使用新的副本数量。

   ```sql
   ALTER TABLE example_db.my_table
   ADD PARTITION p1 VALUES LESS THAN ("2015-01-01")
   ("replication_num"="1");
   ```

4. 修改分区的副本数量。

   ```sql
   ALTER TABLE example_db.my_table
   MODIFY PARTITION p1 SET("replication_num"="1");
   ```

5. 批量修改指定分区的副本数量。

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

1. 基于基础索引 (k1,k2,k3,v1,v2) 创建 Rollup 索引 example_rollup_index，使用列式存储。

   ```sql
   ALTER TABLE example_db.my_table
   ADD ROLLUP example_rollup_index(k1, k3, v1, v2)
   PROPERTIES("storage_type"="column");
   ```

2. 基于 example_rollup_index(k1,k3,v1,v2) 创建索引 example_rollup_index2。

   ```sql
   ALTER TABLE example_db.my_table
   ADD ROLLUP example_rollup_index2 (k1, v1)
   FROM example_rollup_index;
   ```

3. 基于基础索引 (k1, k2, k3, v1) 创建索引 example_rollup_index3，设置 Rollup 超时时间为一小时。

   ```sql
   ALTER TABLE example_db.my_table
   ADD ROLLUP example_rollup_index3(k1, k3, v1)
   PROPERTIES("storage_type"="column", "timeout" = "3600");
   ```

4. 删除索引 example_rollup_index2。

   ```sql
   ALTER TABLE example_db.my_table
   DROP ROLLUP example_rollup_index2;
   ```

### 列

1. 在 example_rollup_index 的 col1 列之后添加非聚合键列 new_col。

   ```sql
   ALTER TABLE example_db.my_table
   ADD COLUMN new_col INT KEY DEFAULT "0" AFTER col1
   TO example_rollup_index;
   ```

2. 在 example_rollup_index 的 col1 列之后添加非聚合值列 new_col。

   ```sql
   ALTER TABLE example_db.my_table
   ADD COLUMN new_col INT DEFAULT "0" AFTER col1
   TO example_rollup_index;
   ```

3. 在 example_rollup_index 的 col1 列之后添加聚合键列 new_col。

   ```sql
   ALTER TABLE example_db.my_table
   ADD COLUMN new_col INT DEFAULT "0" AFTER col1
   TO example_rollup_index;
   ```

4. 在 example_rollup_index 的 col1 列之后添加聚合值列 new_col SUM。

   ```sql
   ALTER TABLE example_db.my_table
   ADD COLUMN new_col INT SUM DEFAULT "0" AFTER col1
   TO example_rollup_index;
   ```

5. 向 example_rollup_index 添加多个聚合列。

   ```sql
   ALTER TABLE example_db.my_table
   ADD COLUMN (col1 INT DEFAULT "1", col2 FLOAT SUM DEFAULT "2.3")
   TO example_rollup_index;
   ```

6. 向 example_rollup_index 添加多个聚合列，并使用 AFTER 指定添加列的位置。

   ```sql
   ALTER TABLE example_db.my_table
   ADD COLUMN col1 INT DEFAULT "1" AFTER `k1`,
   ADD COLUMN col2 FLOAT SUM AFTER `v2`,
   TO example_rollup_index;
   ```

7. 从 example_rollup_index 删除列。

   ```sql
   ALTER TABLE example_db.my_table
   DROP COLUMN col2
   FROM example_rollup_index;
   ```

8. 修改基础索引中 col1 列的类型为 BIGINT，并将其放在 col2 后面。

   ```sql
   ALTER TABLE example_db.my_table
   MODIFY COLUMN col1 BIGINT DEFAULT "1" AFTER col2;
   ```

9. 修改基础索引中 val1 列的最大长度为 64，原长度为 32。

   ```sql
   ALTER TABLE example_db.my_table
   MODIFY COLUMN val1 VARCHAR(64) REPLACE DEFAULT "abc";
   ```

10. 重新排序 example_rollup_index 中的列，原始顺序为 k1、k2、k3、v1、v2。

    ```sql
    ALTER TABLE example_db.my_table
    ORDER BY (k3,k1,k2,v2,v1)
    FROM example_rollup_index;
    ```

11. 同时执行添加列（ADD COLUMN）和排序（ORDER BY）操作。

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

    该操作也可以合并进上述列操作中（注意多条子句的语法略有不同）。

    ```sql
    ALTER TABLE example_db.my_table
    DROP COLUMN col2
    PROPERTIES ("bloom_filter_columns"="k1,k2,k3");
    ```

### 表属性

1. 修改表的共位属性（Colocate）。

   ```sql
   ALTER TABLE example_db.my_table
   SET ("colocate_with" = "t1");
   ```

2. 修改表的动态分区属性。

   ```sql
   ALTER TABLE example_db.my_table
   SET ("dynamic_partition.enable" = "false");
   ```

   如果需要为未配置动态分区属性的表添加动态分区属性，需要指定所有动态分区属性。

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

1. 将 table1 重命名为 table2。

   ```sql
   ALTER TABLE table1 RENAME table2;
   ```

2. 将 example_table 的 Rollup 索引 rollup1 重命名为 rollup2。

   ```sql
   ALTER TABLE example_table RENAME ROLLUP rollup1 rollup2;
   ```

3. 将 example_table 的分区 p1 重命名为 p2。

   ```sql
   ALTER TABLE example_table RENAME PARTITION p1 p2;
   ```

### Bitmap 索引

1. 为 table1 中的列 siteid 创建 Bitmap 索引。

   ```sql
   ALTER TABLE table1
   ADD INDEX index_1 (siteid) [USING BITMAP] COMMENT 'balabala';
   ```

2. 删除 table1 中的 siteid 列的 Bitmap 索引。

   ```sql
   ALTER TABLE table1
   DROP INDEX index_1;
   ```

### 交换

在 table1 和 table2 之间进行原子交换。

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

## 参考资料

- [创建表](./CREATE_TABLE.md)
- [显示创建表](../data-manipulation/SHOW_CREATE_TABLE.md) -
显示创建表的语句
- [显示表格](../data-manipulation/SHOW_TABLES.md) -
-
> 显示表格列表
- [显示更改表](../data-manipulation/SHOW_ALTER.md) -
显示表格更改操作
- [删除表](./DROP_TABLE.md)格
