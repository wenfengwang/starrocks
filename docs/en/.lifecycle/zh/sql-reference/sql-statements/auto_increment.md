---
displayed_sidebar: English
---

# AUTO_INCREMENT

从 3.0 版本开始，StarRocks 支持 `AUTO_INCREMENT` 列属性，可以简化数据管理。本主题介绍了 `AUTO_INCREMENT` 列属性的应用场景、用法和特性。

## 介绍

当将新数据行加载到表中，并且未为 `AUTO_INCREMENT` 列指定值时，StarRocks 会自动为该行的 `AUTO_INCREMENT` 列分配一个整数值，作为该表中该行的唯一ID。`AUTO_INCREMENT` 列的后续值会从该行的ID开始，以特定步长自动增加。`AUTO_INCREMENT` 列可用于简化数据管理，并加快某些查询速度。以下是 `AUTO_INCREMENT` 列的一些应用场景：

- 作为主键：`AUTO_INCREMENT` 列可用作主键，以确保每行都具有唯一的ID，并便于查询和管理数据。
- 表联接：在联接多个表时，`AUTO_INCREMENT` 列可用作联接键，相较于使用数据类型为STRING（例如UUID）的列，它可以加快查询速度。
- 计算高基数列中不同值的数量：`AUTO_INCREMENT` 列可用于表示字典中的唯一值列。与直接计算不同的STRING值相比，有时计算 `AUTO_INCREMENT` 列的不同整数值可以将查询速度提高几倍，甚至是几十倍。

在 CREATE TABLE 语句中，您需要指定一个 `AUTO_INCREMENT` 列。`AUTO_INCREMENT` 列的数据类型必须为BIGINT。`AUTO_INCREMENT` 列的值可以[隐式分配或显式指定](#assign-values-for-auto_increment-column)。它从1开始，每插入一行，值就递增1。

## 基本操作

### 在创建表时指定 `AUTO_INCREMENT` 列

创建一个名为 `test_tbl1` 的表，包含两个列：`id` 和 `number`。将 `number` 列指定为 `AUTO_INCREMENT` 列。

```SQL
CREATE TABLE test_tbl1
(
    id BIGINT NOT NULL, 
    number BIGINT NOT NULL AUTO_INCREMENT
) 
PRIMARY KEY (id) 
DISTRIBUTED BY HASH(id)
PROPERTIES("replicated_storage" = "true");
```

### 为 `AUTO_INCREMENT` 列赋值

#### 隐式赋值

当您将数据加载到StarRocks表中时，无需为 `AUTO_INCREMENT` 列指定值。StarRocks会自动为该列分配唯一的整数值，并将其插入表中。

```SQL
INSERT INTO test_tbl1 (id) VALUES (1);
INSERT INTO test_tbl1 (id) VALUES (2);
INSERT INTO test_tbl1 (id) VALUES (3),(4),(5);
```

查看表中的数据。

```SQL
mysql > SELECT * FROM test_tbl1 ORDER BY id;
+------+--------+
| id   | number |
+------+--------+
|    1 |      1 |
|    2 |      2 |
|    3 |      3 |
|    4 |      4 |
|    5 |      5 |
+------+--------+
5 rows in set (0.02 sec)
```

当您将数据加载到StarRocks表中时，还可以将值指定为 `DEFAULT`，用于 `AUTO_INCREMENT` 列。StarRocks会自动为该列分配唯一的整数值，并将其插入表中。

```SQL
INSERT INTO test_tbl1 (id, number) VALUES (6, DEFAULT);
```

查看表中的数据。

```SQL
mysql > SELECT * FROM test_tbl1 ORDER BY id;
+------+--------+
| id   | number |
+------+--------+
|    1 |      1 |
|    2 |      2 |
|    3 |      3 |
|    4 |      4 |
|    5 |      5 |
|    6 |      6 |
+------+--------+
6 rows in set (0.02 sec)
```

在实际使用中，当您查看表中的数据时，可能会返回以下结果。这是因为StarRocks无法保证 `AUTO_INCREMENT` 列的值严格单调递增。但StarRocks可以保证这些值大致按时间顺序增加。有关详细信息，请参阅[单调性](#monotonicity)。

```SQL
mysql > SELECT * FROM test_tbl1 ORDER BY id;
+------+--------+
| id   | number |
+------+--------+
|    1 |      1 |
|    2 | 100001 |
|    3 | 200001 |
|    4 | 200002 |
|    5 | 200003 |
|    6 | 200004 |
+------+--------+
6 rows in set (0.01 sec)
```

#### 显式指定值

还可以显式指定 `AUTO_INCREMENT` 列的值，并将其插入表中。

```SQL
INSERT INTO test_tbl1 (id, number) VALUES (7, 100);

-- 查看表中的数据。

mysql > SELECT * FROM test_tbl1 ORDER BY id;
+------+--------+
| id   | number |
+------+--------+
|    1 |      1 |
|    2 | 100001 |
|    3 | 200001 |
|    4 | 200002 |
|    5 | 200003 |
|    6 | 200004 |
|    7 |    100 |
+------+--------+
7 rows in set (0.01 sec)
```

此外，显式指定值不会影响StarRocks为新插入的数据行生成的后续值。

```SQL
INSERT INTO test_tbl1 (id) VALUES (8);

-- 查看表中的数据。

mysql > SELECT * FROM test_tbl1 ORDER BY id;
+------+--------+
| id   | number |
+------+--------+
|    1 |      1 |
|    2 | 100001 |
|    3 | 200001 |
|    4 | 200002 |
|    5 | 200003 |
|    6 | 200004 |
|    7 |    100 |
|    8 |      2 |
+------+--------+
8 rows in set (0.01 sec)
```

**注意**

我们建议您不要同时使用隐式分配的值和显式指定的值，用于 `AUTO_INCREMENT` 列。因为指定的值可能与StarRocks生成的值相同，从而破坏了[全局唯一性自增ID](#uniqueness)。

## 基本特性

### 唯一性

一般情况下，StarRocks保证 `AUTO_INCREMENT` 列的值在表中是全局唯一的。我们建议您不要同时隐式分配和显式指定 `AUTO_INCREMENT` 列的值。如果这样做，可能会破坏自增ID的全局唯一性。以下是一个简单示例：创建一个名为 `test_tbl2` 的表，包含两列：`id` 和 `number`。将 `number` 列指定为 `AUTO_INCREMENT` 列。

```SQL
CREATE TABLE test_tbl2
(
    id BIGINT NOT NULL,
    number BIGINT NOT NULL AUTO_INCREMENT
 ) 
PRIMARY KEY (id) 
DISTRIBUTED BY HASH(id)
PROPERTIES("replicated_storage" = "true");
```

在表 `test_tbl2` 中隐式分配并显式指定 `AUTO_INCREMENT` 列 `number` 的值。

```SQL
INSERT INTO test_tbl2 (id, number) VALUES (1, DEFAULT);
INSERT INTO test_tbl2 (id, number) VALUES (2, 2);
INSERT INTO test_tbl2 (id) VALUES (3);
```

查询表 `test_tbl2`。

```SQL
mysql > SELECT * FROM test_tbl2 ORDER BY id;
+------+--------+
| id   | number |
+------+--------+
|    1 |      1 |
|    2 |      2 |
|    3 | 100001 |
+------+--------+
3 rows in set (0.08 sec)
```

### 单调性

为了提高分配自增ID的性能，BE会在本地缓存一些自增ID。在这种情况下，StarRocks无法保证 `AUTO_INCREMENT` 列的值是严格单调递增的。只能确保这些值大致按时间顺序增加。

> **注意**
>
> BE缓存的自增ID数量由FE动态参数 `auto_increment_cache_size` 决定，默认为 `100,000`。您可以使用 `ADMIN SET FRONTEND CONFIG ("auto_increment_cache_size" = "xxx");` 来修改该值。

例如，一个StarRocks集群有一个FE节点和两个BE节点。创建一个名为 `test_tbl3` 的表，并插入五行数据，如下所示：

```SQL
CREATE TABLE test_tbl3
(
    id BIGINT NOT NULL,
    number BIGINT NOT NULL AUTO_INCREMENT
) 
PRIMARY KEY (id)
DISTRIBUTED BY HASH(id)
PROPERTIES("replicated_storage" = "true");

INSERT INTO test_tbl3 VALUES (1, DEFAULT);
INSERT INTO test_tbl3 VALUES (2, DEFAULT);
INSERT INTO test_tbl3 VALUES (3, DEFAULT);

INSERT INTO test_tbl3 VALUES (4, DEFAULT);
INSERT INTO test_tbl3 VALUES (5, DEFAULT);
```

`test_tbl3` 表中的自增 ID 不是单调递增的，因为两个 BE 节点分别缓存了自增 ID 范围为 [1, 100000] 和 [100001, 200000]。当使用多个 INSERT 语句加载数据时，数据会发送到不同的 BE 节点，这些节点会独立分配自增 ID。因此，无法保证自增 ID 是严格单调递增的。

```SQL
mysql > SELECT * FROM test_tbl3 ORDER BY id;
+------+--------+
| id   | number |
+------+--------+
|    1 |      1 |
|    2 | 100001 |
|    3 | 200001 |
|    4 |      2 |
|    5 | 100002 |
+------+--------+
5 rows in set (0.07 sec)
```

## 部分更新和 `AUTO_INCREMENT` 列

本节说明了如何仅更新表中包含的几个指定列，其中包含一个 `AUTO_INCREMENT` 列。

> **注意**
>
> 目前，只有主键表支持部分更新。

### `AUTO_INCREMENT` 列是主键

在进行部分更新时，需要指定主键。因此，如果 `AUTO_INCREMENT` 列是主键或主键的一部分，部分更新的用户行为与未定义 `AUTO_INCREMENT` 列时完全相同。

1. 在数据库 `example_db` 中创建一个名为 `test_tbl4` 的表，并插入一行数据。

    ```SQL
    -- 创建表。
    CREATE TABLE test_tbl4
    (
        id BIGINT AUTO_INCREMENT,
        name BIGINT NOT NULL,
        job1 BIGINT NOT NULL,
        job2 BIGINT NOT NULL
    ) 
    PRIMARY KEY (id, name)
    DISTRIBUTED BY HASH(id)
    PROPERTIES("replicated_storage" = "true");

    -- 准备数据。
    mysql > INSERT INTO test_tbl4 (id, name, job1, job2) VALUES (0, 0, 1, 1);
    Query OK, 1 row affected (0.04 sec)
    {'label':'insert_6af28e77-7d2b-11ed-af6e-02424283676b', 'status':'VISIBLE', 'txnId':'152'}

    -- 查询表。
    mysql > SELECT * FROM test_tbl4 ORDER BY id;
    +------+------+------+------+
    | id   | name | job1 | job2 |
    +------+------+------+------+
    |    0 |    0 |    1 |    1 |
    +------+------+------+------+
    1 row in set (0.01 sec)
    ```

2. 准备名为 **my_data4.csv** 的 CSV 文件，用于更新表 `test_tbl4`。CSV 文件包括 `AUTO_INCREMENT` 列的值，但不包括 `job1` 列的值。第一行的主键已存在于表 `test_tbl4` 中，而第二行的主键不存在于表中。

    ```Plaintext
    0,0,99
    1,1,99
    ```

3. 运行 [Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) 作业，并使用 CSV 文件更新表 `test_tbl4`。

    ```Bash
    curl --location-trusted -u <username>:<password> -H "label:1" \
        -H "column_separator:," \
        -H "partial_update:true" \
        -H "columns:id,name,job2" \
        -T my_data4.csv -XPUT \
        http://<fe_host>:<fe_http_port>/api/example_db/test_tbl4/_stream_load
    ```

4. 查询更新后的表。表中已存在第一行的数据，`test_tbl4`，并且 `job1` 列的值保持不变。第二行的数据是新插入的，因为未指定 `job1` 列的默认值，部分更新框架直接将该列的值设置为 `0`。

    ```SQL
    mysql > SELECT * FROM test_tbl4 ORDER BY id;
    +------+------+------+------+
    | id   | name | job1 | job2 |
    +------+------+------+------+
    |    0 |    0 |    1 |   99 |
    |    1 |    1 |    0 |   99 |
    +------+------+------+------+
    2 rows in set (0.01 sec)
    ```

### `AUTO_INCREMENT` 列不是主键

如果 `AUTO_INCREMENT` 列不是主键或主键的一部分，并且在流加载作业中未提供自增 ID，则会出现以下情况：

- 如果表中已存在该行，StarRocks 不会更新自增 ID。
- 如果该行是新加载到表中，StarRocks 会生成一个新的自增 ID。

此功能可用于构建字典表，以快速计算不同的字符串值。

1. 在数据库 `example_db` 中，创建一个名为 `test_tbl5` 的表，并将列 `job1` 指定为 `AUTO_INCREMENT` 列，并向表 `test_tbl5` 中插入一行数据。

    ```SQL
    -- 创建表。
    CREATE TABLE test_tbl5
    (
        id BIGINT NOT NULL,
        name BIGINT NOT NULL,
        job1 BIGINT NOT NULL AUTO_INCREMENT,
        job2 BIGINT NOT NULL
    )
    PRIMARY KEY (id, name)
    DISTRIBUTED BY HASH(id)
    PROPERTIES("replicated_storage" = "true");

    -- 准备数据。
    mysql > INSERT INTO test_tbl5 VALUES (0, 0, -1, -1);
    Query OK, 1 row affected (0.04 sec)
    {'label':'insert_458d9487-80f6-11ed-ae56-aa528ccd0ebf', 'status':'VISIBLE', 'txnId':'94'}

    -- 查询表。
    mysql > SELECT * FROM test_tbl5 ORDER BY id;
    +------+------+------+------+
    | id   | name | job1 | job2 |
    +------+------+------+------+
    |    0 |    0 |   -1 |   -1 |
    +------+------+------+------+
    1 row in set (0.01 sec)
    ```

2. 准备一个名为 **my_data5.csv** 的 CSV 文件，用于更新表 `test_tbl5`。CSV 文件不包含 `AUTO_INCREMENT` 列 `job1` 的值。表中已存在第一行的主键，而第二行和第三行的主键不存在。

    ```Plaintext
    0,0,99
    1,1,99
    2,2,99
    ```

3. 运行 [Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) 作业，将数据从 CSV 文件加载到表 `test_tbl5` 中。

    ```Bash
    curl --location-trusted -u <username>:<password> -H "label:2" \
        -H "column_separator:," \
        -H "partial_update:true" \
        -H "columns: id,name,job2" \
        -T my_data5.csv -XPUT \
        http://<fe_host>:<fe_http_port>/api/example_db/test_tbl5/_stream_load
    ```

4. 查询更新后的表。表中已存在第一行的数据 `test_tbl5`，因此 `AUTO_INCREMENT` 列 `job1` 保留其原始值。第二行和第三行的数据是新插入的，因此 StarRocks 会为 `AUTO_INCREMENT` 列 `job1` 生成新的值。

    ```SQL
    mysql > SELECT * FROM test_tbl5 ORDER BY id;
    +------+------+--------+------+
    | id   | name | job1   | job2 |
    +------+------+--------+------+
    |    0 |    0 |     -1 |   99 |
    |    1 |    1 |      1 |   99 |
    |    2 |    2 | 100001 |   99 |
    +------+------+--------+------+
    3 rows in set (0.01 sec)
    ```

## 限制

- 创建包含 `AUTO_INCREMENT` 列的表时，必须设置 `'replicated_storage' = 'true'` 以确保所有副本具有相同的自增 ID。
- 每个表只能有一个 `AUTO_INCREMENT` 列。
- `AUTO_INCREMENT` 列的数据类型必须为 BIGINT。
- `AUTO_INCREMENT` 列必须为 `NOT NULL`，并且没有默认值。
- 您可以从包含 `AUTO_INCREMENT` 列的主键表中删除数据。但是，如果 `AUTO_INCREMENT` 列不是主键或主键的一部分，则在以下情况下删除数据时需要注意以下限制：

  - 在执行 DELETE 操作时，还有一个用于部分更新的加载作业，该作业仅包含 UPSERT 操作。如果 UPSERT 和 DELETE 操作都命中同一数据行，并且 UPSERT 操作是在 DELETE 操作之后执行的，则 UPSERT 操作可能不会生效。
  - 在执行包含对同一数据行的多个 UPSERT 和 DELETE 操作的部分更新加载作业时，如果在执行 DELETE 操作后执行了某个 UPSERT 操作，则 UPSERT 操作可能不会生效。
- 不支持使用 ALTER TABLE 添加 `AUTO_INCREMENT` 属性。
- 从 3.1 版本开始，StarRocks 的共享数据模式支持 `AUTO_INCREMENT` 属性。
- StarRocks 不支持指定 `AUTO_INCREMENT` 列的起始值和步长。

## 关键词

AUTO_INCREMENT, AUTO INCREMENT