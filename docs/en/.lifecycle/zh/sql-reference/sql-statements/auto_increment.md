---
displayed_sidebar: English
---

# 自动递增

从3.0版本开始，StarRocks支持`AUTO_INCREMENT`列属性，可以简化数据管理。本节介绍`AUTO_INCREMENT`列属性的应用场景、使用方法和特点。

## 介绍

当新数据行加载到表中并且未指定`AUTO_INCREMENT`列的值时，StarRocks会自动为该行的`AUTO_INCREMENT`列分配一个整数值作为其在表中的唯一ID。`AUTO_INCREMENT`列的后续值从行ID开始按特定步骤自动增加。`AUTO_INCREMENT`列可用于简化数据管理并加快某些查询的速度。以下是`AUTO_INCREMENT`列的一些应用场景：

- 作为主键：可以使用`AUTO_INCREMENT`列作为主键，保证每一行都有唯一的ID，方便查询和管理数据。
- 连接表：连接多个表时，可以使用`AUTO_INCREMENT`列作为Join Key，与使用`STRING`（例如`UUID`）数据类型的列相比，可以加快查询速度。
- 计算高基数列中不同值的数量：`AUTO_INCREMENT`列可用于表示字典中的唯一值列。与直接统计不同的`STRING`值相比，统计`AUTO_INCREMENT`列的不同整数值有时可以提高查询速度数倍甚至数十倍。

您需要在`CREATE TABLE`语句中指定一个`AUTO_INCREMENT`列。`AUTO_INCREMENT`列的数据类型必须是`BIGINT`。`AUTO_INCREMENT`列的值可以[隐式分配或显式指定](#assign-values-for-auto_increment-column)。它从1开始，每添加一个新行就加1。

## 基本操作

### 在创建表时指定`AUTO_INCREMENT`列

创建一个名为`test_tbl1`的表，其中包含两列：`id`和`number`。将`number`列指定为`AUTO_INCREMENT`列。

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

### 为`AUTO_INCREMENT`列分配值

#### 隐式赋值

将数据加载到StarRocks表中时，无需指定`AUTO_INCREMENT`列的值。StarRocks自动为该列分配唯一的整数值并将它们插入到表中。

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
5行在集合中 (0.02秒)
```

当您将数据加载到StarRocks表中时，您还可以将`AUTO_INCREMENT`列的值指定为`DEFAULT`。StarRocks自动为该列分配唯一的整数值并将它们插入到表中。

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
6行在集合中 (0.02秒)
```

在实际使用中，查看数据表时可能会返回以下结果。这是因为StarRocks无法保证`AUTO_INCREMENT`列的值严格单调。但StarRocks可以保证这些值大致按时间顺序增加。有关更多信息，请参阅[单调性](#monotonicity)。

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
6行在集合中 (0.01秒)
```

#### 明确指定值

您还可以显式指定`AUTO_INCREMENT`列的值并将它们插入到表中。

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
7行在集合中 (0.01秒)
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
8行在集合中 (0.01秒)
```

**注意**

我们建议您不要同时对`AUTO_INCREMENT`列使用隐式分配的值和显式指定的值。因为指定的值可能与StarRocks生成的值相同，破坏了[自增ID的全局唯一性](#uniqueness)。

## 基本特点

### 唯一性

一般来说，StarRocks保证`AUTO_INCREMENT`列的值在整个表中是全局唯一的。我们建议您不要同时隐式分配和显式指定`AUTO_INCREMENT`列的值。如果这样做，可能会破坏自动递增ID的全局唯一性。这是一个简单的示例：创建一个名为`test_tbl2`的表，其中包含两列：`id`和`number`。将`number`列指定为`AUTO_INCREMENT`列。

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

隐式分配并显式指定表`test_tbl2`中`AUTO_INCREMENT`列`number`的值。

```SQL
INSERT INTO test_tbl2 (id, number) VALUES (1, DEFAULT);
INSERT INTO test_tbl2 (id, number) VALUES (2, 2);
INSERT INTO test_tbl2 (id) VALUES (3);
```

查询表`test_tbl2`。

```SQL
mysql > SELECT * FROM test_tbl2 ORDER BY id;
+------+--------+
| id   | number |
+------+--------+
|    1 |      1 |
|    2 |      2 |
|    3 | 100001 |
+------+--------+
3行在集合中 (0.08秒)
```

### 单调性

为了提高分配自增ID的性能，BE在本地缓存了部分自增ID。在这种情况下，StarRocks无法保证`AUTO_INCREMENT`列的值严格单调。只能保证数值大致按时间顺序递增。

> **注意**
> BE缓存的自增ID数量由FE动态参数`auto_increment_cache_size`决定，默认为`100,000`。可以通过`ADMIN SET FRONTEND CONFIG ("auto_increment_cache_size" = "xxx");`修改该值

例如，StarRocks集群有1个FE节点和2个BE节点。创建名为`test_tbl3`的表，并插入5行数据，如下：

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
```
# AUTO_INCREMENT

自从3.0版本以来，StarRocks支持`AUTO_INCREMENT`列属性，可以简化数据管理。本主题介绍了`AUTO_INCREMENT`列属性的应用场景、用法和特性。

## 介绍

当将新的数据行加载到表中，并且没有为`AUTO_INCREMENT`列指定值时，StarRocks会自动为该行的`AUTO_INCREMENT`列分配一个整数值作为其在整个表中的唯一ID。`AUTO_INCREMENT`列的后续值会自动从该行的ID开始按照特定步长递增。`AUTO_INCREMENT`列可用于简化数据管理并加快某些查询的速度。以下是`AUTO_INCREMENT`列的一些应用场景：

- 作为主键：`AUTO_INCREMENT`列可用作主键，以确保每行具有唯一的ID，并且便于查询和管理数据。
- 表连接：在多个表连接时，`AUTO_INCREMENT`列可用作连接键，与使用数据类型为STRING（例如UUID）的列相比，可以加快查询速度。
- 计算高基数列中的唯一值数量：`AUTO_INCREMENT`列可用于表示字典中的唯一值列。与直接计算不同的STRING值相比，计算`AUTO_INCREMENT`列的唯一整数值有时可以提高查询速度数倍甚至十几倍。

在CREATE TABLE语句中需要指定`AUTO_INCREMENT`列。`AUTO_INCREMENT`列的数据类型必须是BIGINT。`AUTO_INCREMENT`列的值可以[隐式分配或显式指定](#为AUTO_INCREMENT列分配值)。它从1开始，每插入一行递增1。

## 基本操作

### 在创建表时指定`AUTO_INCREMENT`列

创建一个名为`test_tbl1`的表，包含两列`id`和`number`，并将列`number`指定为`AUTO_INCREMENT`列。

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

### 为`AUTO_INCREMENT`列分配值

#### 隐式分配值

当将数据加载到StarRocks表中时，不需要为`AUTO_INCREMENT`列指定值。StarRocks会自动为该列分配唯一的整数值并插入表中。

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

当将数据加载到StarRocks表中时，还可以将值指定为`DEFAULT`，用于`AUTO_INCREMENT`列。StarRocks会自动为该列分配唯一的整数值并插入表中。

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

在实际使用中，查看表中的数据可能返回以下结果。这是因为StarRocks无法保证`AUTO_INCREMENT`列的值严格单调递增。但是，StarRocks可以保证这些值大致按照时间顺序递增。有关更多信息，请参见[单调性](#单调性)。

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

您还可以显式指定`AUTO_INCREMENT`列的值并将其插入表中。

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

此外，显式指定的值不会影响StarRocks为新插入的数据行生成的后续值。

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

我们建议不要同时使用隐式分配的值和显式指定的值作为`AUTO_INCREMENT`列的值。因为指定的值可能与StarRocks生成的值相同，从而破坏了[自增ID的全局唯一性](#唯一性)。

## 基本特性

### 唯一性

通常情况下，StarRocks保证`AUTO_INCREMENT`列的值在整个表中是全局唯一的。我们建议不要同时隐式分配值和显式指定`AUTO_INCREMENT`列的值。如果这样做，可能会破坏自增ID的全局唯一性。下面是一个简单的示例：创建一个名为`test_tbl2`的表，包含两列`id`和`number`，并将列`number`指定为`AUTO_INCREMENT`列。

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

在表`test_tbl2`中隐式分配和显式指定`AUTO_INCREMENT`列`number`的值。

```SQL
INSERT INTO test_tbl2 (id, number) VALUES (1, DEFAULT);
INSERT INTO test_tbl2 (id, number) VALUES (2, 2);
INSERT INTO test_tbl2 (id) VALUES (3);
```

查询表`test_tbl2`。

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

为了提高分配自增ID的性能，BE节点会在本地缓存一些自增ID。在这种情况下，StarRocks无法保证`AUTO_INCREMENT`列的值是严格单调递增的。只能确保这些值大致按照时间顺序递增。

> **注意**
> BE节点缓存的自增ID数量由FE动态参数`auto_increment_cache_size`决定，默认值为`100,000`。您可以使用`ADMIN SET FRONTEND CONFIG ("auto_increment_cache_size" = "xxx");`修改该值。

例如，一个StarRocks集群有一个FE节点和两个BE节点。创建一个名为`test_tbl3`的表，并插入五行数据，如下所示：

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

表`test_tbl3`中的自增ID不是单调递增的，因为两个BE节点缓存自增ID分别为[1, 100000]和[100001, 200000]。当使用多个INSERT语句加载数据时，数据会被发送到不同的BE节点，BE节点独立分配自增ID。因此，不能保证自增ID是严格单调的。

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

## 限制

- 创建带有`AUTO_INCREMENT`列的表时，必须设置`'replicated_storage' = 'true'`，以确保所有副本具有相同的自增ID。
- 每个表只能有一个`AUTO_INCREMENT`列。
- `AUTO_INCREMENT`列的数据类型必须是BIGINT。
- `AUTO_INCREMENT`列必须为`NOT NULL`，并且没有默认值。
- 您可以从具有`AUTO_INCREMENT`列的主键表中删除数据。但是，如果`AUTO_INCREMENT`列不是主键或主键的一部分，在以下场景删除数据时需要注意以下限制：

  - 在DELETE操作期间，还有一个部分更新的加载作业，其中仅包含UPSERT操作。如果UPSERT和DELETE操作都命中同一数据行，并且在DELETE操作之后执行UPSERT操作，则UPSERT操作可能不生效。
  - 有一个用于部分更新的加载作业，其中包括对同一数据行进行多个UPSERT和DELETE操作。如果在DELETE操作之后执行某个UPSERT操作，则UPSERT操作可能不会生效。

- 不支持使用ALTER TABLE添加`AUTO_INCREMENT`属性。
- 从3.1版本开始，StarRocks的共享数据模式支持`AUTO_INCREMENT`属性。
- StarRocks不支持指定`AUTO_INCREMENT`列的起始值和步长。

## 关键字

AUTO_INCREMENT, AUTO INCREMENT
```markdown
- 不支持使用 ALTER TABLE 添加 `AUTO_INCREMENT` 属性。
- 从 3.1 版本开始，StarRocks 的共享数据模式支持 `AUTO_INCREMENT` 属性。
- StarRocks 不支持指定 `AUTO_INCREMENT` 列的起始值和步长。

## 关键词

`AUTO_INCREMENT`, 自动增量