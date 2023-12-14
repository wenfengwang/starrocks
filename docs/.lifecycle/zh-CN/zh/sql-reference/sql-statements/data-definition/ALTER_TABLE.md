```yaml
   + {R}
   + {R}
 + {R}
+ {R}
```
  ADD COLUMN (column_name1 column_type [KEY | agg_type] DEFAULT "default_value" AFTER (column_name))
  ADD COLUMN (column_name2 column_type [KEY | agg_type] DEFAULT "default_value" AFTER (column_name))
  [TO rollup_index_name]
  [PROPERTIES ("key"="value", ...)]
  ```

注意：

1. If you want to add a value column to the aggregation model, you need to specify the agg_type.
2. If you want to add a key column to the non-aggregation model, you need to specify the KEY keyword.
3. It is not allowed to add columns in the rollup index that already exist in the base index. If needed, you can create a new rollup index.

#### Remove a Column from a Specified Index (DROP COLUMN)

Syntax:

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP COLUMN column_name
[FROM rollup_index_name];
```

Note:

1. It is not allowed to delete partition columns.
2. If you are deleting a column from the base index, and the rollup index contains the column as well, it will also be deleted.

#### Modify the Column Type and Position of a Specified Index (MODIFY COLUMN)

Syntax:

```sql
ALTER TABLE [<db_name>.]<tbl_name>
MODIFY COLUMN column_name column_type [KEY | agg_type] [NULL | NOT NULL] [DEFAULT "default_value"]
[AFTER column_name|FIRST]
[FROM rollup_index_name]
[PROPERTIES ("key"="value", ...)]
```

Note:

1. If you want to modify a value column in the aggregation model, you need to specify the agg_type.
2. If you want to modify a key column in the non-aggregation model, you need to specify the KEY keyword.
3. You can only modify the column type, the other properties of the column remain unchanged (i.e., other properties need to be explicitly specified in the statement according to the original properties, see the 8th example in the [Schema Change](#schema-change-1) section for reference).
4. Partition columns cannot be modified in any way.
5. The following type conversions are currently supported (user must ensure precision loss):

    - TINYINT/SMALLINT/INT/BIGINT can be converted to TINYINT/SMALLINT/INT/BIGINT/DOUBLE.
    - TINTINT/SMALLINT/INT/BIGINT/LARGEINT/FLOAT/DOUBLE/DECIMAL can be converted to VARCHAR.
    - VARCHAR supports modification of maximum length.
    - VARCHAR can be converted to TINTINT/SMALLINT/INT/BIGINT/LARGEINT/FLOAT/DOUBLE.
    - VARCHAR can be converted to DATE (currently supports "%Y-%m-%d", "%y-%m-%d", "%Y%m%d", "%y%m%d", "%Y/%m/%d, "%y/%m/%d " six formatted date formats).
    DATETIME can be converted to DATE (retaining only the year-month-day information, for example: `2019-12-09 21:47:05` &lt;--&gt; `2019-12-09`)
    DATE can be converted to DATETIME (automatic padding of hours, minutes, and seconds, for example: `2019-12-09` &lt;--&gt; `2019-12-09 00:00:00`)
    - FLOAT can be converted to DOUBLE.
    - INT can be converted to DATE (if the INT type data is not valid, the conversion fails and the original data remains unchanged).

6. It is not supported to convert from NULL to NOT NULL.

#### Reorder Columns of a Specified Index

Syntax:

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ORDER BY (column_name1, column_name2, ...)
[FROM rollup_index_name]
[PROPERTIES ("key"="value", ...)]
```

Note:

1. All the columns in the index must be listed.
2. The value column should come after the key column.

#### Add Generated Columns

Syntax:

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ADD COLUMN col_name data_type [NULL] AS generation_expr [COMMENT 'string']
```

Add a generated column and specify the expression used. [Generated Columns](../generated_columns.md) are used to pre-calculate and store the results of an expression, which can accelerate queries containing complex expressions. As of v3.1, StarRocks supports this feature.

#### Modify Table Properties

Support modifying the following table properties:

- `replication_num`
- `default.replication_num`
- `storage_cooldown_ttl`
- `storage_cooldown_time`
- Dynamic partitioning related properties
- `enable_persistent_index`
- `bloom_filter_columns`
- `colocate_with`

Syntax:

```sql
ALTER TABLE [<db_name>.]<tbl_name>
SET ("key" = "value",...)
```

Note: This can also be merged into the above schema change operation to modify the table, see the examples in the [Examples](#示例) section.

### Operate on Rollup Related Syntax

#### Create Rollup Index (ADD ROLLUP)

**RollUp Table Index**: The shortkey index can accelerate data retrieval, but it depends on the order of dimension columns. If non-prefix dimension columns are used to construct query predicates, users can create several RollUp Table indexes for the data table. The data organization and storage of the RollUp Table index are the same as the data table, but the RollUp Table has its own shortkey index. When creating a RollUp Table index, the user can choose the aggregation granularity, the number of columns, and the order of dimension columns, allowing frequently used query conditions to hit the corresponding RollUp Table index.

Syntax:

```SQL
ALTER TABLE [<db_name>.]<tbl_name> 
ADD ROLLUP rollup_name (column_name1, column_name2, ...)
[FROM from_index_name]
[PROPERTIES ("key"="value", ...)];
```

PROPERTIES: Supports setting a timeout period, with the default timeout period being 1 day.

Example:

```SQL
ALTER TABLE [<db_name>.]<tbl_name> 
ADD ROLLUP r1(col1,col2) from r0;
```

#### Batch Create Rollup Index

Syntax:

```SQL
ALTER TABLE [<db_name>.]<tbl_name>
ADD ROLLUP [rollup_name (column_name1, column_name2, ...)
[FROM from_index_name]
[PROPERTIES ("key"="value", ...)],...];
```

Example:

```SQL
ALTER TABLE [<db_name>.]<tbl_name>
ADD ROLLUP r1(col1,col2) from r0, r2(col3,col4) from r0;
```

> Note
>
1. If from_index_name is not specified, it defaults to creating from the base index.
2. The columns in the rollup table must already exist in the from_index.
3. In the properties, it is possible to specify the storage format. Please refer to the [CREATE TABLE](../data-definition/CREATE_TABLE.md) section for details.

#### Delete Rollup Index (DROP ROLLUP)

Syntax:

```SQL
ALTER TABLE [<db_name>.]<tbl_name>
DROP ROLLUP rollup_name [PROPERTIES ("key"="value", ...)];
```

Example:

```sql
ALTER TABLE [<db_name>.]<tbl_name> DROP ROLLUP r1;
```

#### Batch Delete Rollup Index

Syntax:

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP ROLLUP [rollup_name [PROPERTIES ("key"="value", ...)],...];
```

Example:

```sql
ALTER TABLE [<db_name>.]<tbl_name> DROP ROLLUP r1, r2;
```

Note:

It is not allowed to delete the base index.

### Bitmap Index Modification

#### Create Bitmap Index (ADD INDEX)

Syntax:

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ADD INDEX index_name (column [, ...],) [USING BITMAP] [COMMENT 'balabala'];
```

Note:

```plain text
1. Currently only bitmap indexes are supported.
2. BITMAP indexes can only be created on a single column.
```

#### Delete Index (DROP INDEX)

Syntax:

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP INDEX index_name;
```

### Swap Atomic Replacement of Two Tables

Syntax:

```sql
ALTER TABLE [<db_name>.]<tbl_name>
SWAP WITH <tbl_name>;
```

### Manual Compaction (as of version 3.1)

StarRocks uses a Compaction mechanism to merge different data versions that are imported, combining small files into large files, effectively improving query performance.

3.1 版本之前，支持通过两种方式来做 Compaction：

- 系统自动在后台执行 Compaction。Compaction 的粒度是 BE 级，由后台自动执行，用户无法控制具体的数据库或者表。
- 用户通过 HTTP 接口指定 Tablet 来执行 Compaction。

3.1 版本之后，增加了一个 SQL 接口，用户可以通过执行 SQL 命令来手动进行 Compaction，可以指定表、单个或多个分区进行 Compaction。

语法：

```sql
-- 对整张表做 compaction。
ALTER TABLE <tbl_name> COMPACT

-- 指定一个分区进行 compaction。
ALTER TABLE <tbl_name> COMPACT <partition_name>

-- 指定多个分区进行 compaction。
ALTER TABLE <tbl_name> COMPACT (<partition1_name>[,<partition2_name>,...])

-- 对多个分区进行 cumulative compaction。
ALTER TABLE <tbl_name> CUMULATIVE COMPACT (<partition1_name>[,<partition2_name>,...])

-- 对多个分区进行 base compaction。
ALTER TABLE <tbl_name> BASE COMPACT (<partition1_name>[,<partition2_name>,...])
```

执行完 Compaction 后，您可以通过查询 `information_schema` 数据库下的 `be_compactions` 表来查看 Compaction 后的数据版本变化 （`SELECT * FROM information_schema.be_compactions;`）。

## 示例

### table

1. 修改表的默认副本数量，新建分区副本数量默认使用此值。

    ```sql
    ALTER TABLE example_db.my_table
    SET ("default.replication_num" = "2");
    ```

2. 修改单分区表的实际副本数量(只限单分区表)。

    ```sql
    ALTER TABLE example_db.my_table
    SET ("replication_num" = "3");
    ```

3. 修改数据在多副本间的写入和同步方式。

    ```sql
    ALTER TABLE example_db.my_table
    SET ("replicated_storage" = "false");
    ```

    以上示例表示将多副本的写入和同步方式设置为 leaderless replication，即数据同时写入到多个副本，不区分主从副本。更多信息，参见 [CREATE TABLE](CREATE_TABLE.md) 的 `replicated_storage` 参数描述。

### partition

1. 增加分区，现有分区 [MIN, 2013-01-01)，增加分区 [2013-01-01, 2014-01-01)，使用默认分桶方式。

    ```sql
    ALTER TABLE example_db.my_table
    ADD PARTITION p1 VALUES LESS THAN ("2014-01-01");
    ```

2. 增加分区，使用新的分桶数。

    ```sql
    ALTER TABLE example_db.my_table
    ADD PARTITION p1 VALUES LESS THAN ("2015-01-01")
    DISTRIBUTED BY HASH(k1) BUCKETS 20;
    ```

3. 增加分区，使用新的副本数。

    ```sql
    ALTER TABLE example_db.my_table
    ADD PARTITION p1 VALUES LESS THAN ("2015-01-01")
    ("replication_num"="1");
    ```

4. 修改分区副本数。

    ```sql
    ALTER TABLE example_db.my_table
    MODIFY PARTITION p1 SET("replication_num"="1");
    ```

5. 批量修改指定分区。

    ```sql
    ALTER TABLE example_db.my_table
    MODIFY PARTITION (p1, p2, p4) SET("replication_num"="1");
    ```

6. 批量修改所有分区。

    ```sql
    ALTER TABLE example_db.my_table
    MODIFY PARTITION (*) SET("storage_medium"="HDD");
    ```

7. 删除分区。

    ```sql
    ALTER TABLE example_db.my_table
    DROP PARTITION p1;
    ```

8. 增加一个指定上下界的分区。

    ```sql
    ALTER TABLE example_db.my_table
    ADD PARTITION p1 VALUES [("2014-01-01"), ("2014-02-01"));
    ```

### rollup

1. 创建 index `example_rollup_index`，基于 base index（k1, k2, k3, v1, v2），列式存储。

    ```sql
    ALTER TABLE example_db.my_table
    ADD ROLLUP example_rollup_index(k1, k3, v1, v2)
    PROPERTIES("storage_type"="column");
    ```

2. 创建 index `example_rollup_index2`，基于 example_rollup_index（k1, k3, v1, v2）。

    ```sql
    ALTER TABLE example_db.my_table
    ADD ROLLUP example_rollup_index2 (k1, v1)
    FROM example_rollup_index;
    ```

3. 创建 index `example_rollup_index3`，基于 base index (k1, k2, k3, v1), 自定义 rollup 超时时间一小时。

    ```sql
    ALTER TABLE example_db.my_table
    ADD ROLLUP example_rollup_index3(k1, k3, v1)
    PROPERTIES("storage_type"="column", "timeout" = "3600");
    ```

4. 删除 index `example_rollup_index2`。

    ```sql
    ALTER TABLE example_db.my_table
    DROP ROLLUP example_rollup_index2;
    ```

### Schema Change

1. 向 `example_rollup_index` 的 `col1` 后添加一个 key 列 `new_col`（非聚合模型）。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN new_col INT KEY DEFAULT "0" AFTER col1
    TO example_rollup_index;
    ```

2. 向 `example_rollup_index` 的 `col1` 后添加一个 value 列 `new_col`（非聚合模型）。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN new_col INT DEFAULT "0" AFTER col1
    TO example_rollup_index;
    ```

3. 向 `example_rollup_index` 的 `col1` 后添加一个 key 列 `new_col`（聚合模型）。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN new_col INT DEFAULT "0" AFTER col1
    TO example_rollup_index;
    ```

4. 向 `example_rollup_index` 的 `col1` 后添加一个 value 列 `new_col`（SUM 聚合类型）（聚合模型）。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN new_col INT SUM DEFAULT "0" AFTER col1
    TO example_rollup_index;
    ```

5. 向 `example_rollup_index` 添加多列（聚合模型）。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN (col1 INT DEFAULT "1", col2 FLOAT SUM DEFAULT "2.3")
    TO example_rollup_index;
    ```

6. 向 `example_rollup_index` 添加多列并通过 `AFTER` 指定列的添加位置。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN col1 INT DEFAULT "1" AFTER `k1`,
    ADD COLUMN col2 FLOAT SUM AFTER `v2`,
    TO example_rollup_index;
    ```

7. 从 `example_rollup_index` 删除一列。

    ```sql
    ALTER TABLE example_db.my_table
    DROP COLUMN col2
    FROM example_rollup_index;
    ```

8. 修改 base index 的 `col1` 列的类型为 BIGINT，并移动到 `col2` 列后面。

    ```sql
    ALTER TABLE example_db.my_table
    MODIFY COLUMN col1 BIGINT DEFAULT "1" AFTER col2;
    ```

9. 修改 base index 的 `val1` 列最大长度。原 `val1` 为 (`val1 VARCHAR(32) REPLACE DEFAULT "abc"`).

    ```sql
    ALTER TABLE example_db.my_table
    MODIFY COLUMN val1 VARCHAR(64) REPLACE DEFAULT "abc";
    ```

10. 重新排序 `example_rollup_index` 中的列（设原列顺序为：k1, k2, k3, v1, v2）。

    ```sql
    ALTER TABLE example_db.my_table
    ORDER BY (k3,k1,k2,v2,v1)
    FROM example_rollup_index;
    ```

11. 同时执行两种操作。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN v2 INT MAX DEFAULT "0" AFTER k2 TO example_rollup_index,
    ORDER BY (k3,k1,k2,v2,v1) FROM example_rollup_index;
```

12. Modify the bloom filter column of the table.

    ```sql
    ALTER TABLE example_db.my_table
    SET ("bloom_filter_columns"="k1,k2,k3");
    ```

    It can also be combined with the above schema change operation (note the slight difference in syntax for multiple clauses)

    ```sql
    ALTER TABLE example_db.my_table
    DROP COLUMN col2
    PROPERTIES ("bloom_filter_columns"="k1,k2,k3");
    ```

13. Modify the Colocate property of the table.

    ```sql
    ALTER TABLE example_db.my_table
    SET ("colocate_with" = "t1");
    ```

14. Change the table's bucketing from Random Distribution to Hash Distribution.

    ```sql
    ALTER TABLE example_db.my_table
    SET ("distribution_type" = "hash");
    ```

15. Modify the dynamic partition properties of the table (support adding dynamic partition properties to tables that have not added dynamic partition properties).

    ```sql
    ALTER TABLE example_db.my_table
    SET ("dynamic_partition.enable" = "false");
    ```

    If you need to add dynamic partition properties to a table that has not added dynamic partition properties, you need to specify all dynamic partition properties.

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

### rename

1. Rename the table `table1` to `table2`.

    ```sql
    ALTER TABLE table1 RENAME table2;
    ```

2. Modify the rollup index named `rollup1` in table `example_table` to `rollup2`.

    ```sql
    ALTER TABLE example_table RENAME ROLLUP rollup1 rollup2;
    ```

3. Modify the partition named `p1` in table `example_table` to `p2`.

    ```sql
    ALTER TABLE example_table RENAME PARTITION p1 p2;
    ```

### index

1. Create a `bitmap` index for `siteid` on `table1`.

    ```sql
    ALTER TABLE table1
    ADD INDEX index_name (siteid) [USING BITMAP] COMMENT 'balabala';
    ```

2. Drop the bitmap index of the `siteid` column on `table1`.

    ```sql
    ALTER TABLE table1 DROP INDEX index_name;
    ```

### swap

Swap `table1` with `table2` atomically.

```sql
ALTER TABLE table1 SWAP WITH table2;
```

### Manual Compaction Example

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

## Related References

- [CREATE TABLE](./CREATE_TABLE.md)
- [SHOW CREATE TABLE](../data-manipulation/SHOW_CREATE_TABLE.md)
- [SHOW TABLES](../data-manipulation/SHOW_TABLES.md)
- [SHOW ALTER TABLE](../data-manipulation/SHOW_ALTER.md)
- [DROP TABLE](./DROP_TABLE.md)