---
displayed_sidebar: English
---

# 生成列

从 v3.1 开始，StarRocks 支持生成列。生成列可用于加速具有复杂表达式的查询。此功能支持预计算和存储表达式和[查询重写](#query-rewrites)的结果，从而显著加快了具有相同复杂表达式的查询速度。

您可以在创建表时定义一个或多个生成列，以存储表达式的结果。因此，当执行包含表达式的查询时，其结果存储在您定义的生成列中，CBO 将重写查询以直接从生成列中读取数据。或者，您可以直接查询生成列中的数据。

此外，还建议**评估生成列对加载性能的影响，因为计算表达式需要一些时间**。此外，建议**[在创建表时创建生成列](#create-generated-columns-at-table-creation-recommended)，而不是在创建表后添加或修改它们**。因为在创建表后添加或修改生成列既费时又费钱。

## 基本操作

### 创建生成列

#### 语法

```SQL
<col_name> <data_type> [NULL] AS <expr> [COMMENT 'string']
```

#### 在创建表时创建生成列（推荐）

创建一个名为 `test_tbl1` 的表，该表包含五列，其中列 `newcol1` 和 `newcol2` 是生成列，其值是通过使用指定的表达式计算的，并分别引用常规列 `data_array` 和 `data_json` 的值。

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
- 聚合函数不能在生成列的表达式中使用。
- 生成列的表达式不能引用其他生成列或[自增列](./auto_increment.md)，但表达式可以引用多个常规列。
- 生成列的数据类型必须与生成列的表达式生成的结果的数据类型匹配。
- 无法在聚合表上创建生成列。
- 目前，StarRocks 的共享数据模式不支持生成列。

#### 创建表后添加生成列

> **注意**
>
> 此操作既费时又耗费资源。因此，建议在创建表时添加生成列。如果不可避免地使用 ALTER TABLE 添加生成列，建议提前评估所涉及的成本和时间。

1. 创建一个名为 `test_tbl2` 的表，该表包含三个常规列 `id`、`data_array` 和 `data_json`。向表中插入数据行。

    ```SQL
    -- 创建表
    CREATE TABLE test_tbl2
    (
        id INT NOT NULL,
        data_array ARRAY<int> NOT NULL,
        data_json JSON NOT NULL
    )
    PRIMARY KEY (id)
    DISTRIBUTED BY HASH(id);

    -- 插入数据行
    INSERT INTO test_tbl2 VALUES (1, [1,2], parse_json('{"a" : 1, "b" : 2}'));

    -- 查询表
    MySQL [example_db]> select * from test_tbl2;
    +------+------------+------------------+
    | id   | data_array | data_json        |
    +------+------------+------------------+
    |    1 | [1,2]      | {"a": 1, "b": 2} |
    +------+------------+------------------+
    1 row in set (0.04 sec)
    ```

2. 使用 ALTER TABLE ... ADD COLUMN ... 添加生成列 `newcol1` 和 `newcol2`，这些列是通过根据常规列 `data_array` 和 `data_json` 的值计算表达式而创建的。

    ```SQL
    ALTER TABLE test_tbl2
    ADD COLUMN newcol1 DOUBLE AS array_avg(data_array);

    ALTER TABLE test_tbl2
    ADD COLUMN newcol2 String AS json_string(json_query(data_json, "$.a"));
    ```

    **注意**：

    - 不支持将生成列添加到聚合表。
    - 必须在生成列之前定义常规列。当您使用 ALTER TABLE ... ADD COLUMN ... 语句添加新的常规列而不指定新常规列的位置时，系统会自动将其放在生成列之前。此外，您不能使用 AFTER 明确将常规列放在生成列之后。

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

    结果显示生成列 `newcol1` 和 `newcol2` 被添加到表中，并且 StarRocks 会根据表达式自动计算它们的值。

### 将数据加载到生成列中

在数据加载过程中，StarRocks 会根据表达式自动计算生成列的值。不能指定生成列的值。以下示例使用 [INSERT INTO](../../loading/InsertInto.md) 语句加载数据：

1. 使用 INSERT INTO 将记录插入 `test_tbl1` 表中。请注意，在 `VALUES ()` 子句中不能指定生成列的值。

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

    结果显示，StarRocks 会根据表达式自动计算生成列 `newcol1` 和 `newcol2` 的值。

    **注意**：

    如果在数据加载期间为生成列指定值，则会返回以下错误：

    ```SQL
    MySQL [example_db]> INSERT INTO test_tbl1 (id, data_array, data_json, newcol1, newcol2) 
    VALUES (2, [3,4], parse_json('{"a" : 3, "b" : 4}'), 3.5, "3");
    ERROR 1064 (HY000): Getting analyzing error. Detail message: materialized column 'newcol1' can not be specified.

    MySQL [example_db]> INSERT INTO test_tbl1 VALUES (2, [3,4], parse_json('{"a" : 3, "b" : 4}'), 3.5, "3");
    ERROR 1064 (HY000): Getting analyzing error. Detail message: Column count doesn't match value count.
    ```

### 修改生成列

> **注意**
>
> 此操作既费时又耗费资源。如果不可避免地使用 ALTER TABLE 来修改生成列，建议提前评估所涉及的成本和时间。

您可以修改生成列的数据类型和表达式。

1. 创建一个名为 `test_tbl3` 的表，该表包含五列，其中列 `newcol1` 和 `newcol2` 是生成列，其值是通过使用指定的表达式计算的，并分别引用常规列 `data_array` 和 `data_json` 的值。在表中插入数据行。

    ```SQL
    -- 创建表
    MySQL [example_db]> CREATE TABLE test_tbl3
    (
        id INT NOT NULL,
        data_array ARRAY<int> NOT NULL,
        data_json JSON NOT NULL,
        -- The data types and expressions of generated columns are specified as follows:
        newcol1 DOUBLE AS array_avg(data_array),
        newcol2 String AS json_string(json_query(data_json, "$.a"))
    )
    PRIMARY KEY (id)
    DISTRIBUTED BY HASH(id);

    -- 插入数据行
    INSERT INTO test_tbl3 (id, data_array, data_json)
    VALUES (1, [1,2], parse_json('{"a" : 1, "b" : 2}'));

    -- 查询表

    MySQL [example_db]> select * from test_tbl3;
    +------+------------+------------------+---------+---------+
    | id   | data_array | data_json        | newcol1 | newcol2 |
    +------+------------+------------------+---------+---------+
    |    1 | [1,2]      | {"a": 1, "b": 2} |     1.5 | 1       |
    +------+------------+------------------+---------+---------+
    1 row in set (0.01 sec)
    ```

2. 修改了生成的列 `newcol1` 和 `newcol2`：

    - 将生成列 `newcol1` 的数据类型更改为 `ARRAY<INT>`，并将其表达式更改为 `data_array`。

        ```SQL
        ALTER TABLE test_tbl3 
        MODIFY COLUMN newcol1 ARRAY<INT> AS data_array;
        ```

    - 修改生成列 `newcol2` 的表达式，从正则列 `data_json` 中提取字段 `b` 的值。

        ```SQL
        ALTER TABLE test_tbl3
        MODIFY COLUMN newcol2 String AS json_string(json_query(data_json, "$.b"));
        ```

3. 查看修改后的架构和表中的数据。

    - 查看修改后的架构。

        ```SQL
        MySQL [example_db]> show create table test_tbl3\G
        **** 1. row ****
            Table: test_tbl3
        Create Table: CREATE TABLE test_tbl3 (
        id int(11) NOT NULL COMMENT "",
        data_array array<int(11)> NOT NULL COMMENT "",
        data_json json NOT NULL COMMENT "",
        -- 修改后，生成列的数据类型和表达式如下：
        newcol1 array<int(11)> NULL AS example_db.test_tbl3.data_array COMMENT "",
        newcol2 varchar(65533) NULL AS json_string(json_query(example_db.test_tbl3.data_json, '$.b')) COMMENT ""
        ) ENGINE=OLAP 
        PRIMARY KEY(id)
        DISTRIBUTED BY HASH(id)
        PROPERTIES (...);
        1 row in set (0.00 sec)
        ```

    - 修改后查询表数据。结果显示，StarRocks 根据修改后的表达式重新计算生成列 `newcol1` 和 `newcol2` 的值。

        ```SQL
        MySQL [example_db]> select * from test_tbl3;
        +------+------------+------------------+---------+---------+
        | id   | data_array | data_json        | newcol1 | newcol2 |
        +------+------------+------------------+---------+---------+
        |    1 | [1,2]      | {"a": 1, "b": 2} | [1,2]   | 2       |
        +------+------------+------------------+---------+---------+
        1 row in set (0.01 sec)
        ```

### 删除生成的列

 从表 `test_tbl3` 中删除 `newcol1` 列。

```SQL
ALTER TABLE test_tbl3 DROP COLUMN newcol1;
```

> **注意**：
>
> 如果生成的列引用了表达式中的常规列，则不能直接删除或修改该常规列。相反，您需要先删除生成的列，然后删除或修改常规列。

### 查询重写

如果查询中的表达式与生成的列的表达式匹配，则优化程序会自动重写查询以直接读取生成的列的值。

1. 假设您使用以下架构创建表 `test_tbl4`：

    ```SQL
    CREATE TABLE test_tbl4
    (
        id INT NOT NULL,
        data_array ARRAY<int> NOT NULL,
        data_json JSON NOT NULL,
        newcol1 DOUBLE AS array_avg(data_array),
        newcol2 String AS json_string(json_query(data_json, "$.a"))
    )
    PRIMARY KEY (id) DISTRIBUTED BY HASH(id);
    ```

2. 如果使用语句 `SELECT array_avg(data_array), json_string(json_query(data_json, "$.a")) FROM test_tbl4;` 查询表 `test_tbl4` 中的数据，则查询仅涉及常规列 `data_array` 和 `data_json`。但是，查询中的表达式与生成的列 `newcol1` 和 `newcol2` 的表达式匹配。在本例中，执行计划显示 CBO 会自动重写查询以读取生成的列 `newcol1` 和 `newcol2` 的值。

    ```SQL
    MySQL [example_db]> EXPLAIN SELECT array_avg(data_array), json_string(json_query(data_json, "$.a")) FROM test_tbl4;
    +---------------------------------------+
    | Explain String                        |
    +---------------------------------------+
    | PLAN FRAGMENT 0                       |
    |  OUTPUT EXPRS:4: newcol1 | 5: newcol2 | -- 查询已重写以读取生成的列 newcol1 和 newcol2 的值。
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
    15 rows in set (0.00 sec)
    ```

### 部分更新和生成的列

若要对主键表执行部分更新，必须在参数中指定生成的列引用的所有常规列 `columns` 。以下示例使用 Stream Load 执行部分更新。

1. 创建一个 `test_tbl5` 包含五列的表，其中列 `newcol1` 和 `newcol2` 是生成列，其值是使用指定的表达式计算的，并分别引用常规列 `data_array` 和 `data_json` 的值。在表中插入数据行。

    ```SQL
    -- 创建表。
    CREATE TABLE test_tbl5
    (
        id INT NOT NULL,
        data_array ARRAY<int> NOT NULL,
        data_json JSON NULL,
        newcol1 DOUBLE AS array_avg(data_array),
        newcol2 String AS json_string(json_query(data_json, "$.a"))
    )
    PRIMARY KEY (id)
    DISTRIBUTED BY HASH(id);

    -- 插入数据行。
    INSERT INTO test_tbl5 (id, data_array, data_json)
    VALUES (1, [1,2], parse_json('{"a" : 1, "b" : 2}'));

    -- 查询表。
    MySQL [example_db]> select * from test_tbl5;
    +------+------------+------------------+---------+---------+
    | id   | data_array | data_json        | newcol1 | newcol2 |
    +------+------------+------------------+---------+---------+
    |    1 | [1,2]      | {"a": 1, "b": 2} |     1.5 | 1       |
    +------+------------+------------------+---------+---------+
    1 row in set (0.01 sec)
    ```

2. 准备一个 CSV 文件 `my_data1.csv` 以更新表 `test_tbl5` 的某些列。

    ```SQL
    1|[3,4]|{"a": 3, "b": 4}
    2|[3,4]|{"a": 3, "b": 4} 
    ```

3. 使用 [Stream Load](../../loading/StreamLoad.md) 与文件 `my_data1.csv` 一起，以更新表 `test_tbl5` 的某些列。您需要在参数中设置 `partial_update:true` 并指定生成的列引用的所有常规列 `columns` 。

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
    [example_db]> select * from test_tbl5;
    +------+------------+------------------+---------+---------+
    | id   | data_array | data_json        | newcol1 | newcol2 |
    +------+------------+------------------+---------+---------+
    |    1 | [3,4]      | {"a": 3, "b": 4} |     3.5 | 3       |
    |    2 | [3,4]      | {"a": 3, "b": 4} |     3.5 | 3       |
    +------+------------+------------------+---------+---------+
    2 rows in set (0.01 sec)
    ```

如果在未指定生成的列引用的所有常规列的情况下执行部分更新，则流加载会返回错误。

1. 准备一个 CSV 文件 `my_data2.csv`。

      ```csv
      1|[3,4]
      2|[3,4]
      ```

2. 当使用[流加载](../../loading/StreamLoad.md)处理`my_data2.csv`文件进行部分列更新时，如果`my_data2.csv`中未提供`data_json`列的数值，并且流加载作业中的`columns`参数不包括`data_json`列，即使`data_json`列允许为空值，Stream Load也会因为生成的`newcol2`列引用了`data_json`列而返回错误。