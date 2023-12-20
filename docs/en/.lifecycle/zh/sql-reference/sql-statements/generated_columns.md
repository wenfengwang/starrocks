---
displayed_sidebar: English
---

# 生成列

自 v3.1 起，StarRocks 支持生成列。生成列可用于加速包含复杂表达式的查询。该功能支持表达式的预计算和存储以及[查询改写](#query-rewrites)，显著提升了包含相同复杂表达式的查询速度。

您可以在创建表时定义一个或多个生成列来存储表达式的结果。因此，当执行包含已存储在您定义的生成列中的表达式的查询时，CBO 会重写查询，直接从生成列读取数据。您也可以直接查询生成列中的数据。

同时建议**评估生成列对加载性能的影响，因为计算表达式需要一定时间**。此外，建议**[在创建表时创建生成列](#create-generated-columns-at-table-creation-recommended)**，而不是在表创建后添加或修改它们，因为在表创建后添加或修改生成列**耗时且成本高**。

## 基本操作

### 创建生成列

#### 语法

```SQL
<col_name> <data_type> [NULL] AS <expr> [COMMENT 'string']
```

#### 在创建表时创建生成列（推荐）

创建一个名为 `test_tbl1` 的表，包含五个列，其中 `newcol1` 和 `newcol2` 是生成列，它们的值通过指定的表达式计算得出，并分别引用常规列 `data_array` 和 `data_json` 的值。

```SQL
CREATE TABLE test_tbl1
(
    id INT NOT NULL,
    data_array ARRAY<int> NOT NULL,
    data_json JSON NOT NULL,
    newcol1 DOUBLE AS array_avg(data_array),
    newcol2 String AS json_string(json_query(data_json, "$.a"))
)
PRIMARY KEY (id)
DISTRIBUTED BY HASH(id);
```

**注意**：

- 生成列必须在常规列之后定义。
- 聚合函数不能用于生成列的表达式中。
- 生成列的表达式不能引用其他生成列或[自增列](./auto_increment.md)，但可以引用多个常规列。
- 生成列的数据类型必须与该生成列表达式的结果数据类型相匹配。
- 无法在聚合表上创建生成列。
- 目前，StarRocks 的共享数据模式不支持生成列。

#### 在表创建后添加生成列

> **注意**
> 此操作耗时且资源密集。因此，建议在创建表时添加生成列。如果不得不使用 **ALTER TABLE** 来添加生成列，建议提前评估所涉及的成本和时间。

1. 创建一个名为 `test_tbl2` 的表，包含三个常规列 `id`、`data_array` 和 `data_json`。向表中插入一条数据行。

   ```SQL
   -- 创建表。
   CREATE TABLE test_tbl2
   (
       id INT NOT NULL,
       data_array ARRAY<int> NOT NULL,
       data_json JSON NOT NULL
   )
   PRIMARY KEY (id)
   DISTRIBUTED BY HASH(id);
   
   -- 插入数据行。
   INSERT INTO test_tbl2 VALUES (1, [1,2], parse_json('{"a" : 1, "b" : 2}'));
   
   -- 查询表。
   MySQL [example_db]> SELECT * FROM test_tbl2;
   +------+------------+------------------+
   | id   | data_array | data_json        |
   +------+------------+------------------+
   |    1 | [1,2]      | {"a": 1, "b": 2} |
   +------+------------+------------------+
   1 row in set (0.04 sec)
   ```

2. 执行 `ALTER TABLE ... ADD COLUMN ...` 来添加生成列 `newcol1` 和 `newcol2`，这些列通过评估基于常规列 `data_array` 和 `data_json` 的值的表达式创建。

   ```SQL
   ALTER TABLE test_tbl2
   ADD COLUMN newcol1 DOUBLE AS array_avg(data_array);
   
   ALTER TABLE test_tbl2
   ADD COLUMN newcol2 String AS json_string(json_query(data_json, "$.a"));
   ```

   **注意**：

   - 不支持向聚合表添加生成列。
   - 常规列需要在生成列之前定义。使用 `ALTER TABLE ... ADD COLUMN ...` 语句添加常规列时，如果没有指定新常规列的位置，系统会自动将其放在生成列之前。此外，不能使用 `AFTER` 显式地将常规列放在生成列之后。

3. 查询表数据。

   ```SQL
   MySQL [example_db]> SELECT * FROM test_tbl2;
   +------+------------+------------------+---------+---------+
   | id   | data_array | data_json        | newcol1 | newcol2 |
   +------+------------+------------------+---------+---------+
   |    1 | [1,2]      | {"a": 1, "b": 2} |     1.5 | 1       |
   +------+------------+------------------+---------+---------+
   1 row in set (0.04 sec)
   ```

   结果显示，生成列 `newcol1` 和 `newcol2` 已添加到表中，并且 StarRocks 根据表达式自动计算它们的值。

### 将数据加载到生成列中

在数据加载过程中，StarRocks 会自动计算生成列的值，基于表达式。您不能指定生成列的值。以下示例使用 [INSERT INTO](../../loading/InsertInto.md) 语句加载数据：

1. 使用 `INSERT INTO` 向 `test_tbl1` 表插入一条记录。注意，您不能在 `VALUES ()` 子句中指定生成列的值。

   ```SQL
   INSERT INTO test_tbl1 (id, data_array, data_json)
   VALUES (1, [1,2], parse_json('{"a" : 1, "b" : 2}'));
   ```

2. 查询表数据。

   ```SQL
   MySQL [example_db]> SELECT * FROM test_tbl1;
   +------+------------+------------------+---------+---------+
   | id   | data_array | data_json        | newcol1 | newcol2 |
   +------+------------+------------------+---------+---------+
   |    1 | [1,2]      | {"a": 1, "b": 2} |     1.5 | 1       |
   +------+------------+------------------+---------+---------+
   1 row in set (0.01 sec)
   ```

   结果显示，StarRocks 根据表达式自动计算了生成列 `newcol1` 和 `newcol2` 的值。

   **注意**：

   如果在数据加载期间指定了生成列的值，则会返回以下错误：

   ```SQL
   MySQL [example_db]> INSERT INTO test_tbl1 (id, data_array, data_json, newcol1, newcol2) 
   VALUES (2, [3,4], parse_json('{"a" : 3, "b" : 4}'), 3.5, "3");
   ERROR 1064 (HY000): Getting analyzing error. Detail message: materialized column 'newcol1' cannot be specified.
   
   MySQL [example_db]> INSERT INTO test_tbl1 VALUES (2, [3,4], parse_json('{"a" : 3, "b" : 4}'), 3.5, "3");
   ERROR 1064 (HY000): Getting analyzing error. Detail message: Column count doesn't match value count.
   ```

### 修改生成列

> **注意**
> 此操作耗时且资源密集。如果不得不使用 `ALTER TABLE` 来修改生成列，建议提前评估所涉及的成本和时间。

您可以修改生成列的数据类型和表达式。

1. 创建一个名为 `test_tbl3` 的表，包含五个列，其中 `newcol1` 和 `newcol2` 是生成列，它们的值通过指定的表达式计算得出，并分别引用常规列 `data_array` 和 `data_json` 的值。向表中插入一条数据行。

   ```SQL
   -- 创建表。
   MySQL [example_db]> CREATE TABLE test_tbl3
   (
       id INT NOT NULL,
       data_array ARRAY<int> NOT NULL,
       data_json JSON NOT NULL,
       -- 生成列的数据类型和表达式如下所指定：
       newcol1 DOUBLE AS array_avg(data_array),
       newcol2 String AS json_string(json_query(data_json, "$.a"))
   )
   PRIMARY KEY (id)
   DISTRIBUTED BY HASH(id);
   
   -- 插入数据行。
   INSERT INTO test_tbl3 (id, data_array, data_json)
```
```SQL
   VALUES (1, [1,2], parse_json('{"a" : 1, "b" : 2}'));
   
   -- 查询表格。
   MySQL [example_db]> SELECT * FROM test_tbl3;
   +------+------------+------------------+---------+---------+
   | id   | data_array | data_json        | newcol1 | newcol2 |
   +------+------------+------------------+---------+---------+
   |    1 | [1,2]      | {"a": 1, "b": 2} |     1.5 | 1       |
   +------+------------+------------------+---------+---------+
   1 行在集合中 (0.01 秒)
   ```

2. 修改生成的列 `newcol1` 和 `newcol2`：

-     将生成列 `newcol1` 的数据类型更改为 `ARRAY<INT>`，并将其表达式更改为 `data_array`。

      ```SQL
      ALTER TABLE test_tbl3 
      MODIFY COLUMN newcol1 ARRAY<INT> AS data_array;
      ```

-     修改生成列 `newcol2` 的表达式，以从常规列 `data_json` 中提取字段 `b` 的值。

      ```SQL
      ALTER TABLE test_tbl3
      MODIFY COLUMN newcol2 STRING AS json_string(json_query(data_json, "$.b"));
      ```

3. 查看修改后的架构和表中的数据。

-     查看修改后的架构。

      ```SQL
      MySQL [example_db]> SHOW CREATE TABLE test_tbl3\G
      **** 1. row ****
          Table: test_tbl3
      Create Table: CREATE TABLE test_tbl3 (
      id INT NOT NULL COMMENT "",
      data_array ARRAY<INT> NOT NULL COMMENT "",
      data_json JSON NOT NULL COMMENT "",
      -- 修改后，生成列的数据类型和表达式如下：
      newcol1 ARRAY<INT> NULL AS example_db.test_tbl3.data_array COMMENT "",
      newcol2 VARCHAR(65533) NULL AS json_string(json_query(example_db.test_tbl3.data_json, '$.b')) COMMENT ""
      ) ENGINE=OLAP 
      PRIMARY KEY(id)
      DISTRIBUTED BY HASH(id)
      PROPERTIES (...);
      1 行在集合中 (0.00 秒)
      ```

-     查询修改后的表数据。结果显示，StarRocks 根据修改后的表达式重新计算了生成列 `newcol1` 和 `newcol2` 的值。

      ```SQL
      MySQL [example_db]> SELECT * FROM test_tbl3;
      +------+------------+------------------+---------+---------+
      | id   | data_array | data_json        | newcol1 | newcol2 |
      +------+------------+------------------+---------+---------+
      |    1 | [1,2]      | {"a": 1, "b": 2} | [1,2]   | 2       |
      +------+------------+------------------+---------+---------+
      1 行在集合中 (0.01 秒)
      ```

### 删除生成列

从表 `test_tbl3` 中删除列 `newcol1`

```SQL
ALTER TABLE test_tbl3 DROP COLUMN newcol1;
```

> **注意**：
> 如果生成列在表达式中引用了常规列，则不能直接删除或修改该常规列。相反，您需要首先删除生成列，然后才能删除或修改常规列。

### 查询重写

如果查询中的表达式与生成列的表达式匹配，优化器会自动重写查询，直接读取生成列的值。

1. 假设您使用以下架构创建了表 `test_tbl4`：

   ```SQL
   CREATE TABLE test_tbl4
   (
       id INT NOT NULL,
       data_array ARRAY<INT> NOT NULL,
       data_json JSON NOT NULL,
       newcol1 DOUBLE AS array_avg(data_array),
       newcol2 STRING AS json_string(json_query(data_json, "$.a"))
   )
   PRIMARY KEY (id) DISTRIBUTED BY HASH(id);
   ```

2. 如果您使用 `SELECT array_avg(data_array), json_string(json_query(data_json, "$.a")) FROM test_tbl4;` 语句查询表 `test_tbl4` 中的数据，查询将仅涉及常规列 `data_array` 和 `data_json`。但是，查询中的表达式与生成列 `newcol1` 和 `newcol2` 的表达式匹配。在这种情况下，执行计划显示 CBO 自动重写查询，读取生成列 `newcol1` 和 `newcol2` 的值。

   ```SQL
   MySQL [example_db]> EXPLAIN SELECT array_avg(data_array), json_string(json_query(data_json, "$.a")) FROM test_tbl4;
   +---------------------------------------+
   | Explain String                        |
   +---------------------------------------+
   | PLAN FRAGMENT 0                       |
   |  OUTPUT EXPRS:4: newcol1 | 5: newcol2 | -- 查询被重写以从生成列 newcol1 和 newcol2 读取数据。
   |   PARTITION: RANDOM                   |
   |                                       |
   |   RESULT SINK                         |
   |                                       |
   |   0:OlapScanNode                      |
   |      TABLE: test_tbl4                 |
   |      PREAGGREGATION: ON               |
   |      partitions=0/1                   |
   |      rollup: test_tbl4                |
   |      tabletRatio=0/0                  |
   |      tabletList=                      |
   |      cardinality=1                    |
   |      avgRowSize=2.0                   |
   +---------------------------------------+
   15 行在集合中 (0.00 秒)
   ```

### 部分更新和生成列

要对主键表执行部分更新，您必须在 `columns` 参数中指定生成列引用的所有常规列。以下示例使用 Stream Load 执行部分更新。

1. 创建一个表 `test_tbl5`，包含 5 列，其中 `newcol1` 和 `newcol2` 是生成列，其值是使用指定的表达式计算得到的，并分别引用常规列 `data_array` 和 `data_json` 的值。向表中插入一条数据行。

   ```SQL
   -- 创建表格。
   CREATE TABLE test_tbl5
   (
       id INT NOT NULL,
       data_array ARRAY<INT> NOT NULL,
       data_json JSON NULL,
       newcol1 DOUBLE AS array_avg(data_array),
       newcol2 STRING AS json_string(json_query(data_json, "$.a"))
   )
   PRIMARY KEY (id)
   DISTRIBUTED BY HASH(id);
   
   -- 插入数据行。
   INSERT INTO test_tbl5 (id, data_array, data_json)
   VALUES (1, [1,2], parse_json('{"a" : 1, "b" : 2}'));
   
   -- 查询表格。
   MySQL [example_db]> SELECT * FROM test_tbl5;
   +------+------------+------------------+---------+---------+
   | id   | data_array | data_json        | newcol1 | newcol2 |
   +------+------------+------------------+---------+---------+
   |    1 | [1,2]      | {"a": 1, "b": 2} |     1.5 | 1       |
   +------+------------+------------------+---------+---------+
   1 行在集合中 (0.01 秒)
   ```

2. 准备 CSV 文件 `my_data1.csv`，用于更新 `test_tbl5` 表中的某些列。

   ```SQL
   1|[3,4]|{"a": 3, "b": 4}
   2|[3,4]|{"a": 3, "b": 4} 
   ```

3. 使用 [Stream Load](../../loading/StreamLoad.md) 和文件 `my_data1.csv` 来更新 `test_tbl5` 表的某些列。您需要设置 `partial_update:true` 并在 `columns` 参数中指定所有被生成列引用的常规列。

   ```Bash
   curl --location-trusted -u <username>:<password> -H "label:1" \
       -H "column_separator:|" \
       -H "partial_update:true" \
       -H "columns:id,data_array,data_json" \ 
       -T my_data1.csv -XPUT \
       http://<fe_host>:<fe_http_port>/api/example_db/test_tbl5/_stream_load
   ```

4. 查询表数据。

   ```SQL
   [example_db]> SELECT * FROM test_tbl5;
   +------+------------+------------------+---------+---------+
   | id   | data_array | data_json        | newcol1 | newcol2 |
   +------+------------+------------------+---------+---------+
   |    1 | [3,4]      | {"a": 3, "b": 4} |     3.5 | 3       |
   |    2 | [3,4]      | {"a": 3, "b": 4} |     3.5 | 3       |
   +------+------------+------------------+---------+---------+
   2 行在集合中 (0.01 秒)
   ```

如果在执行部分更新时未指定生成列引用的所有常规列，则 Stream Load 会返回错误。

1. 准备 CSV 文件 `my_data2.csv`。

   ```csv
   1|[3,4]
   2|[3,4]
   ```
```markdown
2. 当使用 [Stream Load](../../loading/StreamLoad.md) 和 `my_data2.csv` 文件进行部分列更新时，如果 `my_data2.csv` 中没有提供 `data_json` 列的值，并且 Stream Load 作业中的 `columns` 参数没有包括 `data_json` 列，即使 `data_json` 列允许空值，由于 `data_json` 列被生成列 `newcol2` 引用，Stream Load 也会返回错误。