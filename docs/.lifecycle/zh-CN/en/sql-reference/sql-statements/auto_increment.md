---
displayed_sidebar: "Chinese"
---

# AUTO_INCREMENT

从 3.0 版本开始，StarRocks 支持 `AUTO_INCREMENT` 列属性，可以简化数据管理。本主题介绍了 `AUTO_INCREMENT` 列属性的应用场景、用法和特性。

## 简介

当加载新数据行到表中并且没有为 `AUTO_INCREMENT` 列指定值时，StarRocks 会自动为该行的 `AUTO_INCREMENT` 列分配一个整数值作为表内唯一标识。`AUTO_INCREMENT` 列的后续值会从该行的 ID 开始按照特定步长自增。`AUTO_INCREMENT` 列可用于简化数据管理并加速某些查询。以下是 `AUTO_INCREMENT` 列的一些应用场景：

- 作为主键：`AUTO_INCREMENT` 列可用作主键，以确保每行具有唯一的标识并且易于查询和管理数据。
- 连接表：在多个表连接时，`AUTO_INCREMENT` 列可用作连接键，相对于使用数据类型为 STRING（例如 UUID）的列，能加速查询。
- 计算高基数列中的唯一值数量：`AUTO_INCREMENT` 列可用于表示字典中的唯一值列。相较于直接计算 DISTINCT STRING 值，计算 `AUTO_INCREMENT` 列的唯一整数值有时可以提高查询速度数倍甚至数十倍。

在 `CREATE TABLE` 语句中需要为 `AUTO_INCREMENT` 列指定。`AUTO_INCREMENT` 列的数据类型必须为 `BIGINT`。`AUTO_INCREMENT` 列的值可以【隐式分配或显式指定】(#assign-values-for-auto_increment-column)。其起始值为 1，每加载新行时自增 1。

## 基本操作

### 在表创建时指定 `AUTO_INCREMENT` 列

创建名为 `test_tbl1` 的表，包含 `id` 和 `number` 两列。将列 `number` 指定为 `AUTO_INCREMENT` 列。

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

### 为 `AUTO_INCREMENT` 列分配值

#### 隐式分配值

当您将数据加载到 StarRocks 表中时，无需为 `AUTO_INCREMENT` 列指定值。StarRocks 会自动为该列分配唯一的整数值并将其插入到表中。

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

当您将数据加载到 StarRocks 表中时，还可以将值指定为 `DEFAULT` 以用于 `AUTO_INCREMENT` 列。StarRocks 会自动为该列分配唯一的整数值并将其插入到表中。

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

在实际使用中，查看表中的数据可能会返回以下结果。这是因为 StarRocks 无法保证 `AUTO_INCREMENT` 列的值严格单调递增。但 StarRocks 可以保证这些值大致按时间顺序递增。有关更多信息，请参阅【单调性】(#monotonicity)。

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

您还可以显式为 `AUTO_INCREMENT` 列指定值并将其插入到表中。

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

此外，显式指定值不会影响 StarRocks 为新插入的数据行生成的后续值。

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

我们建议您不要同时使用隐式分配值和显式指定值为 `AUTO_INCREMENT` 列。因为指定的值可能与 StarRocks 生成的值相同，从而违反了【全局唯一性】(#uniqueness)。

## 基本特性

### 唯一性

一般来说，StarRocks 保证 `AUTO_INCREMENT` 列的值在表级别上是全局唯一的。我们建议您不要同时隐式分配和显式指定 `AUTO_INCREMENT` 列的值。如果这样做，可能会破坏自增 ID 的全局唯一性。以下是一个简单的示例：创建名为 `test_tbl2` 的表，包含 `id` 和 `number` 两列。将列 `number` 指定为 `AUTO_INCREMENT` 列。

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

在表 `test_tbl2` 中同时隐式分配和显式指定 `AUTO_INCREMENT` 列 `number` 的值。

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

为了提高分配自增 ID 的性能，BE 本地缓存了一些自增 ID。在此情况下，StarRocks 无法保证 `AUTO_INCREMENT` 列的值严格单调递增。只能确保这些值大致按时间顺序递增。

> **注意**
>
> BE 缓存的自增 ID 数量由 FE 动态参数 `auto_increment_cache_size` 决定，默认为 `100,000`。您可以通过使用 `ADMIN SET FRONTEND CONFIG ("auto_increment_cache_size" = "xxx");` 修改该值。

例如，一个 StarRocks 集群拥有一个 FE 节点和两个 BE 节点。创建名为 `test_tbl3` 的表，并插入五行数据如下：

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

表`test_tbl3`中的自增ID不是单调递增的，因为两个BE节点缓存了自增ID，分别为[1, 100000]和[100001, 200000]。当使用多个INSERT语句加载数据时，数据被发送到不同的BE节点，这些节点独立分配自增ID。因此，不能保证自增ID是严格单调递增的。

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

## 部分更新和`AUTO_INCREMENT`列

本节说明如何仅更新包含`AUTO_INCREMENT`列的表中的少数指定列。

> **注意**
>
> 目前，仅支持主键表的部分更新。

### `AUTO_INCREMENT`列是主键

在部分更新中，需要指定主键。因此，如果`AUTO_INCREMENT`列是主键或主键的一部分，那么用户对部分更新的行为与未定义`AUTO_INCREMENT`列时的行为完全相同。

1. 在数据库`example_db`中创建表`test_tbl4`并插入一行数据。

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

2. 准备CSV文件**my_data4.csv**以更新表`test_tbl4`。CSV文件包括`AUTO_INCREMENT`列的值，并不包括`job1`列的值。第一行的主键已经存在于表`test_tbl4`中，而第二行的主键在表中不存在。

    ```Plaintext
    0,0,99
    1,1,99
    ```

3. 运行[Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)作业，使用CSV文件更新表`test_tbl4`。

    ```Bash
    curl --location-trusted -u <username>:<password> -H "label:1" \
        -H "column_separator:," \
        -H "partial_update:true" \
        -H "columns:id,name,job2" \
        -T my_data4.csv -XPUT \
        http://<fe_host>:<fe_http_port>/api/example_db/test_tbl4/_stream_load
    ```

4. 查询更新后的表。数据的第一行已经存在于表`test_tbl4`中，列`job1`的值保持不变。第二行的数据是新插入的，因为未指定列`job1`的默认值，部分更新框架直接将该列的值设置为`0`。

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

### `AUTO_INCREMENT`列不是主键

如果`AUTO_INCREMENT`列不是主键或主键的一部分，并且在Stream Load作业中未提供自增ID，则会出现以下情况：

- 如果行已经存在于表中，StarRocks不会更新`AUTO_INCREMENT`ID。
- 如果行是新加载到表中的，StarRocks会生成一个新的`AUTO_INCREMENT`ID。

此功能可用于构建用于快速计算不同字符串值的字典表。

1. 在数据库`example_db`中，创建表`test_tbl5`并指定列`job1`为`AUTO_INCREMENT`列，并向表`test_tbl5`插入一行数据。

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

2. 准备CSV文件**my_data5.csv**以更新表`test_tbl5`。CSV文件不包含`AUTO_INCREMENT`列`job1`的值。第一行的主键已经存在于表`test_tbl5`中，而第二行和第三行的主键在表中不存在。

    ```Plaintext
    0,0,99
    1,1,99
    2,2,99
    ```

3. 运行[Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)作业，从CSV文件加载数据到表`test_tbl5`中。

    ```Bash
    curl --location-trusted -u <username>:<password> -H "label:2" \
        -H "column_separator:," \
        -H "partial_update:true" \
        -H "columns: id,name,job2" \
        -T my_data5.csv -XPUT \
        http://<fe_host>:<fe_http_port>/api/example_db/test_tbl5/_stream_load
    ```

4. 查询更新后的表。数据的第一行已经存在于表`test_tbl5`中，因此`AUTO_INCREMENT`列`job1`保持原始值。第二行和第三行的数据是新插入的，因此StarRocks为`AUTO_INCREMENT`列`job1`生成了新的值。

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

- 创建带有`AUTO_INCREMENT`列的表时，必须设置`'replicated_storage' = 'true'`，以确保所有副本具有相同的自增ID。
- 每个表只能有一个`AUTO_INCREMENT`列。
- `AUTO_INCREMENT`列的数据类型必须是BIGINT。
- `AUTO_INCREMENT`列必须为`NOT NULL`，并且没有默认值。
- 您可以从带有`AUTO_INCREMENT`列的主键表中删除数据。但是，如果`AUTO_INCREMENT`列不是主键或主键的一部分，当您在以下情况中删除数据时，需要注意以下限制：

  - 在进行DELETE操作期间，也会有一个用于部分更新的加载作业，该作业仅包含UPSERT操作。如果UPSERT和DELETE操作同时命中相同的数据行，并且在DELETE操作之后执行了UPSERT操作，那么UPSERT操作可能不会生效。
  - 存在一个用于部分更新的加载作业，该作业会在同一数据行上执行多个UPSERT和DELETE操作。如果在DELETE操作之后执行特定的UPSERT操作，则UPSERT操作可能不会生效。

- 通过使用ALTER TABLE添加`AUTO_INCREMENT`属性是不被支持的。
- 自3.1版本起，StarRocks的共享数据模式支持`AUTO_INCREMENT`属性。
- StarRocks不支持为`AUTO_INCREMENT`列指定起始值和步长。