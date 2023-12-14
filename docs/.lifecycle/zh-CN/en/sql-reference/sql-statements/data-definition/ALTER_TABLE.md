---
displayed_sidebar: "Chinese"
---

# ALTER TABLE（修改表）

## 描述

修改现有表，包括：

- [重命名表、分区、索引](#重命名)
- [修改表注释](#自v3.1起修改表注释)
- [原子交换](#交换)
- [添加/删除分区和修改分区属性](#修改分区)
- [模式更改](#模式更改)
- [创建/删除滚动索引](#修改滚动索引)
- [修改位图索引](#修改位图索引)
- [手动数据版本压缩](#自v3.1起手动压缩)

> **注意**
>
> 此操作需要在目标表上具有 ALTER 权限。

## 语法

```SQL
ALTER TABLE [<db_name>.]<tbl_name>
alter_clause1[, alter_clause2, ...]
```

`alter_clause` 分为六种操作：partition、rollup、模式更改、rename、index、swap、comment 和 compact。

- 重命名：重命名表、滚动索引或分区。**注意无法修改列名。**
- comment：修改表注释（从**v3.1**开始支持）。
- swap：原子交换两个表。
- partition：修改分区属性，删除分区或添加分区。
- 模式更改：添加、删除或重新排序列，或修改列类型。
- rollup：创建或删除滚动索引。
- index：修改索引（仅支持修改位图索引）。
- compact：执行手动压缩以合并加载数据的版本（从**v3.1**开始支持）。

:::note

- 无法在一个 ALTER TABLE 语句中执行模式更改、滚动和分区操作。
- 模式更改和滚动是异步操作。任务提交后立即返回成功消息。您可以运行 [SHOW ALTER TABLE](../data-manipulation/SHOW_ALTER.md) 命令来检查进度。
- 分区、重命名、交换和索引是同步操作，命令返回表示执行已完成。
:::

### 重命名

重命名支持修改表名、滚动索引和分区名。

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

### 自v3.1起修改表注释

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name> COMMENT = "<新表注释>";
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
3. 如果未指定 bucket 模式，则将自动使用内置表使用的 bucket 方法。
4. 如果指定了 bucket 模式，只能修改 bucket 数，并且无法修改 bucket 模式或 bucket 列。
5. 用户可以在 `["key"="value"]` 中设置分区的一些属性。详细信息请参见 [CREATE TABLE](CREATE_TABLE.md)。

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

1. 保留至少一个分区以供分区表使用。
2. 执行 DROP PARTITION 一段时间后，可以通过 RECOVER 语句恢复已删除的分区。详细信息请参见 RECOVER 语句。
3. 如果执行 DROP PARTITION FORCE，分区将被直接删除，未完成的活动将无法恢复。因此，通常不建议执行此操作。

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

- 分区的以下属性可被修改：

  - storage_medium
  - storage_cooldown_ttl 或 storage_cooldown_time
  - replication_num

- 对于仅具有一个分区的表，分区名称与表名称相同。如果表分为多个分区，可以使用 `(*)` 来修改所有分区的属性，这更加方便。

- 执行 `SHOW PARTITIONS FROM <tbl_name>` 以查看修改后的分区属性。

### 模式更改

模式更改支持以下修改。

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

1. 如果向聚合表添加值列，需要指定 agg_type。
2. 如果向非聚合表（如重复键表）添加键列，需要指定 KEY 关键字。
3. 无法将已存在于基础索引中的列添加到滚动索引中。（如果需要，可以重新创建滚动索引。）

#### 在指定索引上添加多列

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
  ADD COLUMN (column_name1 column_type [KEY | agg_type] DEFAULT "default_value" AFTER (column_name))
  ADD COLUMN (column_name2 column_type [KEY | agg_type] DEFAULT "default_value" AFTER (column_name))
  [TO rollup_index_name]
  [PROPERTIES ("key"="value", ...)]
  ```

注意：

1. 如果向聚合表添加值列，需要指定 `agg_type`。

2. 如果向非聚合表添加键列，需要指定 `KEY` 关键字。

3. 无法将已存在于基础索引中的列添加到滚动索引中。（如果需要，可以创建另一个滚动索引。）

#### 从指定索引中删除列

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP COLUMN column_name
[FROM rollup_index_name];
```

注意：

1. 无法删除分区列。
2. 如果从基础索引中删除列，如果该列包含在滚动索引中，则该列也会被删除。

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

1. 如果在聚合模型中修改值列，需要指定 `agg_type`。
2. 如果在非聚合模型中修改键列，需要指定 `KEY` 关键字。
3. 仅能修改列的类型。列的其他属性保持不变（即根据原始属性在语句中需要明确书写其他属性，参见示例 8）。
4. 无法修改分区列。
5. 目前支持以下类型的转换（用户可保证精度损失）。

   - 将 TINYINT/SMALLINT/INT/BIGINT 转换为 TINYINT/SMALLINT/INT/BIGINT/DOUBLE。
   - 将 TINYINT/SMALLINT/INT/BIGINT/LARGEINT/FLOAT/DOUBLE/DECIMAL 转换为 VARCHAR。VARCHAR 支持修改最大长度。
   - 将 VARCHAR 转换为 TINYINT/SMALLINT/INT/BIGINT/LARGEINT/FLOAT/DOUBLE。
   - 将 VARCHAR 转换为 DATE（目前支持六种格式：“%Y-%m-%d”，“%y-%m-%d”，“%Y%m%d”，“%y%m%d”，“%Y/%m/%d”，“%y/%m/%d”）
   - 将 DATETIME 转换为 DATE（仅保留年-月-日信息，即 `2019-12-09 21:47:05` `<-->` `2019-12-09`）
   - 将 DATE 转换为 DATETIME（将时、分、秒设置为零，例如： `2019-12-09` `<-->` `2019-12-09 00:00:00`）
   - 将 FLOAT 转换为 DOUBLE
   - 将 INT 转换为 DATE（如果 INT 数据转换失败，则保留原始数据）

6. 不支持从 NULL 转换为 NOT NULL。

#### 重新指定索引的列顺序

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ORDER BY (column_name1, column_name2, ...)
[FROM rollup_index_name]
[PROPERTIES ("key"="value", ...)]
```

注意：

1. 必须写入索引中的所有列。
2. 在关键列之后列出值列。

#### 添加生成的列（从 v3.1 版本开始）

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ADD COLUMN col_name data_type [NULL] AS generation_expr [COMMENT 'string']
```

您可以添加一个生成的列，并指定其表达式。[生成的列](../generated_columns.md) 可用于预先计算和存储表达式的结果，从而显著加速具有相同复杂表达式的查询。从 v3.1 版本开始，StarRocks 支持生成的列。

#### 修改表属性

目前，StarRocks 支持修改以下表属性：

- `replication_num`
- `default.replication_num`
- `storage_cooldown_ttl`
- `storage_cooldown_time`
- 与动态分区相关的属性
- `enable_persistent_index`
- `bloom_filter_columns`
- `colocate_with`

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
SET ("key" = "value",...)
```

注意：
您也可以通过合并上述模式更改操作来修改属性。请参阅以下示例。

### 修改 rollup 索引

#### 创建一个 rollup 索引

语法：

```SQL
ALTER TABLE [<db_name>.]<tbl_name> 
ADD ROLLUP rollup_name (column_name1, column_name2, ...)
[FROM from_index_name]
[PROPERTIES ("key"="value", ...)]
```

属性：支持设置超时时间，默认超时时间为一天。

示例：

```SQL
ALTER TABLE [<db_name>.]<tbl_name> 
ADD ROLLUP r1(col1,col2) from r0;
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
ADD ROLLUP r1(col1,col2) from r0, r2(col3,col4) from r0;
```

注意：

1. 如果未指定 from_index_name，则默认从基本索引创建。
2. rollup 表中的列必须是 from_index 中的现有列。
3. 在属性中，用户可以指定存储格式。有关详细信息，请参见 CREATE TABLE。

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

```plain text
1. 位图索引仅适用于当前版本。
2. 仅在单个列中创建 BITMAP 索引。
```

#### 删除索引

语法：

```sql
DROP INDEX index_name;
```

### 交换

交换支持原子交换两个表。

语法：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
SWAP WITH <tbl_name>;
```

### 手动压实（从 3.1 版本开始）

StarRocks 使用压实机制合并加载的不同版本数据。此功能可以将小文件合并为大文件，从而有效提高查询性能。

在 v3.1 之前，压实有两种方式：

- 系统自动压实：在后台的 BE 级别执行压实。用户无法为压实指定数据库或表。
- 用户可以通过调用 HTTP 接口执行压实。

从 v3.1 开始，StarRocks 提供了 SQL 接口，供用户通过运行 SQL 命令手动执行压实。他们可以选择特定的表或分区进行压实。这样可以更灵活地控制压实过程。

语法：

```sql
-- 对整个表进行压实。
ALTER TABLE <tbl_name> COMPACT

-- 对单个分区进行压实。
ALTER TABLE <tbl_name> COMPACT <partition_name>

-- 对多个分区进行压实。
ALTER TABLE <tbl_name> COMPACT (<partition1_name>[,<partition2_name>,...])

-- 执行累积压实。
ALTER TABLE <tbl_name> CUMULATIVE COMPACT (<partition1_name>[,<partition2_name>,...])

-- 执行基本压实。
ALTER TABLE <tbl_name> BASE COMPACT (<partition1_name>[,<partition2_name>,...])
```

`information_schema` 数据库中的 `be_compactions` 表记录压实结果。您可以运行 `SELECT * FROM information_schema.be_compactions;` 来查询压实后的数据版本。

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

3. 修改副本之间的数据写入和复制模式。

    ```sql
    ALTER TABLE example_db.my_table
    SET ("replicated_storage" = "false");
    ```

   该示例将数据写入和副本之间的数据写入和复制模式设置为“无主副本复制”，即意味着数据同时写入多个副本，而不区分主副本和次级副本。有关更多信息，请参见 [CREATE TABLE](CREATE_TABLE.md) 中的 `replicated_storage` 参数。

### 分区

1. 添加一个分区并使用默认的分桶模式。现有分区为 [MIN, 2013-01-01)。添加的分区为 [2013-01-01, 2014-01-01)。

    ```sql
    ALTER TABLE example_db.my_table
    ADD PARTITION p1 VALUES LESS THAN ("2014-01-01");
    ```

2. 添加一个分区并使用新的分桶数。

    ```sql
    ALTER TABLE example_db.my_table
    ADD PARTITION p1 VALUES LESS THAN ("2015-01-01")
    DISTRIBUTED BY HASH(k1);
    ```

3. 添加一个分区并使用新的副本数。

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

6. 批量更改所有分区的存储介质。

    ```sql
    ALTER TABLE example_db.my_table
    MODIFY PARTITION (*) SET("storage_medium"="HDD");
    ```

7. 删除一个分区。

    ```sql
    ALTER TABLE example_db.my_table
    DROP PARTITION p1;
    ```

8. 添加一个具有上下边界的分区。

    ```sql
    ALTER TABLE example_db.my_table
    ADD PARTITION p1 VALUES [("2014-01-01"), ("2014-02-01"));
    ```

### 滚动聚集

1. 基于基本索引(k1，k2，k3，v1，v2)创建索引`example_rollup_index`。采用基于列的存储。

    ```sql
    ALTER TABLE example_db.my_table
    ADD ROLLUP example_rollup_index(k1, k3, v1, v2)
    PROPERTIES("storage_type"="column");
    ```

2. 基于`example_rollup_index(k1，k3，v1，v2)`创建索引`example_rollup_index2`。

    ```sql
    ALTER TABLE example_db.my_table
    ADD ROLLUP example_rollup_index2 (k1, v1)
    FROM example_rollup_index;
    ```

3. 基于基本索引(k1，k2，k3，v1)创建索引`example_rollup_index3`。聚集超时时间设为一小时。

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

### 模式更改

1. 在`example_rollup_index`的`col1`列后添加一个键列`new_col`（非聚合列）。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN new_col INT KEY DEFAULT "0" AFTER col1
    TO example_rollup_index;
    ```

2. 在`example_rollup_index`的`col1`列后添加一个值列`new_col`（非聚合列）。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN new_col INT DEFAULT "0" AFTER col1
    TO example_rollup_index;
    ```

3. 在`example_rollup_index`的`col1`列后添加一个键列`new_col`（聚合列）。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN new_col INT DEFAULT "0" AFTER col1
    TO example_rollup_index;
    ```

4. 在`example_rollup_index`的`col1`列后添加一个值列`new_col SUM`（聚合列）。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN new_col INT SUM DEFAULT "0" AFTER col1
    TO example_rollup_index;
    ```

5. 向`example_rollup_index`（聚合）添加多个列。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN (col1 INT DEFAULT "1", col2 FLOAT SUM DEFAULT "2.3")
    TO example_rollup_index;
    ```

6. 向`example_rollup_index`（聚合）添加多个列，并使用`AFTER`指定添加列的位置。

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

8. 将基本索引的`col1`列类型修改为BIGINT，并放在`col2`列之后。

    ```sql
    ALTER TABLE example_db.my_table
    MODIFY COLUMN col1 BIGINT DEFAULT "1" AFTER col2;
    ```

9. 将基本索引的`val1`列的最大长度修改为64。原始长度为32。

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

11. 一次执行两个操作（ADD COLUMN和ORDER BY）。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN v2 INT MAX DEFAULT "0" AFTER k2 TO example_rollup_index,
    ORDER BY (k3,k1,k2,v2,v1) FROM example_rollup_index;
    ```

12. 更改表的bloomfilter列。

     ```sql
     ALTER TABLE example_db.my_table
     SET ("bloom_filter_columns"="k1,k2,k3");
     ```

     该操作也可以合并到上述模式更改操作中（请注意，多个子句的语法略有不同）。

     ```sql
     ALTER TABLE example_db.my_table
     DROP COLUMN col2
     PROPERTIES ("bloom_filter_columns"="k1,k2,k3");
     ```

13. 更改表的Colocate属性。

     ```sql
     ALTER TABLE example_db.my_table
     SET ("colocate_with" = "t1");
     ```

14. 将表的分桶模式从随机分布更改为哈希分布。

     ```sql
     ALTER TABLE example_db.my_table
     SET ("distribution_type" = "hash");
     ```

15. 更改表的动态分区属性。

     ```sql
     ALTER TABLE example_db.my_table
     SET ("dynamic_partition.enable" = "false");
     ```

     如果需要向未配置动态分区属性的表添加动态分区属性，则需要指定所有动态分区属性。

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

2. 将`example_table`中的滚动索引`rollup1`重命名为`rollup2`。

    ```sql
    ALTER TABLE example_table RENAME ROLLUP rollup1 rollup2;
    ```

3. 将`example_table`中的分区`p1`重命名为`p2`。

    ```sql
    ALTER TABLE example_table RENAME PARTITION p1 p2;
    ```

### 索引

1. 为`table1`中的`siteid`列创建位图索引。

    ```sql
    ALTER TABLE table1
    ADD INDEX index_1 (siteid) [USING BITMAP] COMMENT 'balabala';
    ```

2. 删除`table1`中的`siteid`列的位图索引。

    ```sql
    ALTER TABLE table1
    DROP INDEX index_1;
    ```

### 交换

`table1`和`table2`之间的原子交换。

```sql
ALTER TABLE table1 SWAP WITH table2
```

### 手动整理示例

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

- [CREATE TABLE](./CREATE_TABLE.md)
- [SHOW CREATE TABLE](../data-manipulation/SHOW_CREATE_TABLE.md)
- [SHOW TABLES](../data-manipulation/SHOW_TABLES.md)
- [SHOW ALTER TABLE](../data-manipulation/SHOW_ALTER.md)
- [删除表](./DELETE_TABLE.md)