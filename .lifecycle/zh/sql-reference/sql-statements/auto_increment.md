---
displayed_sidebar: English
---

# 自动递增

从3.0版本开始，StarRocks支持AUTO_INCREMENT列属性，可简化数据管理。本文介绍了AUTO_INCREMENT列属性的应用场景、使用方法和特性。

## 介绍

当新数据行载入表中且未指定AUTO_INCREMENT列的值时，StarRocks会自动为该行的AUTO_INCREMENT列分配一个整数值，作为其在表中的唯一ID。AUTO_INCREMENT列的后续值将从行ID开始，按特定步长自动递增。AUTO_INCREMENT列可用于简化数据管理，并加快某些查询。以下是AUTO_INCREMENT列的一些应用场景：

- 作为主键：AUTO_INCREMENT列可用作主键，确保每行都有唯一ID，便于查询和数据管理。
- 连接表：在多表连接时，AUTO_INCREMENT列可作为连接键（Join Key），相比使用字符串类型的列（如UUID），可加快查询速度。
- 计算高基数列的不同值数量：AUTO_INCREMENT列可用于代表字典中的唯一值列。与直接统计不同字符串值相比，统计AUTO_INCREMENT列的不同整数值有时可将查询速度提高数倍甚至数十倍。

您需要在CREATE TABLE语句中指定`AUTO_INCREMENT`列。`AUTO_INCREMENT`列的数据类型必须是BIGINT。AUTO_INCREMENT列的值可以[隐式分配或显式指定](#assign-values-for-auto_increment-column)。其起始值为1，每新增一行数据则增加1。

## 基本操作

### 在创建表时指定AUTO_INCREMENT列

创建一个名为test_tbl1的表，包含两列：id和number。将number列指定为AUTO_INCREMENT列。

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

### 为AUTO_INCREMENT列分配值

#### 隐式赋值

当您将数据加载到StarRocks表时，无需指定AUTO_INCREMENT列的值。StarRocks会自动为该列分配唯一的整数值并插入表中。

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

在将数据加载到StarRocks表时，您也可以为AUTO_INCREMENT列指定DEFAULT值。StarRocks将自动为该列分配唯一的整数值并插入表中。

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

在实际使用中，查看表中的数据时可能会返回以下结果。这是因为StarRocks无法保证`AUTO_INCREMENT`列的值是严格单调递增的。但StarRocks可以保证这些值大致按时间顺序递增。更多信息，请参见 [单调性](#monotonicity)。

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

您也可以显式指定AUTO_INCREMENT列的值并将其插入表中。

```SQL
INSERT INTO test_tbl1 (id, number) VALUES (7, 100);

-- view data in the table.

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

此外，显式指定值不会影响StarRocks为新插入数据行生成的后续值。

```SQL
INSERT INTO test_tbl1 (id) VALUES (8);

-- view data in the table.

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

建议不要同时使用`AUTO_INCREMENT`列的隐式赋值和显式指定值。因为所指定的值可能与StarRocks生成的值相同，这会破坏[自增ID的全局唯一性](#uniqueness)。

## 基本特性

### 唯一性

通常，StarRocks保证AUTO_INCREMENT列的值在整个表中全局唯一。建议不要同时使用隐式赋值和显式指定AUTO_INCREMENT列的值。如果这样做，可能会破坏自增ID的全局唯一性。以下是一个简单的示例：创建一个名为test_tbl2的表，包含两列：id和number。将number列指定为AUTO_INCREMENT列。

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

在test_tbl2表中隐式赋值并显式指定AUTO_INCREMENT列的值。

```SQL
INSERT INTO test_tbl2 (id, number) VALUES (1, DEFAULT);
INSERT INTO test_tbl2 (id, number) VALUES (2, 2);
INSERT INTO test_tbl2 (id) VALUES (3);
```

查询test_tbl2表。

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

为了提高分配自增ID的性能，BE会在本地缓存一些自增ID。因此，StarRocks无法保证AUTO_INCREMENT列的值严格单调递增。它只能确保这些值大致按时间顺序递增。

> **注意**
> BE缓存的自增ID数量由FE动态参数`auto_increment_cache_size`决定，默认值为`100,000`。您可以通过`ADMIN SET FRONTEND CONFIG ("auto_increment_cache_size" = "xxx")`来修改该值。

例如，一个StarRocks集群有一个FE节点和两个BE节点。创建一个名为test_tbl3的表并插入如下五行数据：

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

由于两个BE节点分别缓存了[1, 100000]和[100001, 200000]的自增ID，test_tbl3表中的自增ID并非严格单调递增。当通过多个INSERT语句加载数据时，数据会被发送到不同的BE节点，而这些BE节点会独立分配自增ID。因此，无法保证自增ID的严格单调性。

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

## 部分更新和AUTO_INCREMENT列

本节解释了如何仅更新包含AUTO_INCREMENT列的表中的特定几列。

> **注意**
> 目前，只有**主键表**支持**部分更新**。

### AUTO_INCREMENT列作为主键

在部分更新时，需要指定主键。因此，如果AUTO_INCREMENT列是主键或主键的一部分，用户在部分更新时的操作与未定义AUTO_INCREMENT列时完全相同。

1. 在example_db数据库中创建一个名为test_tbl4的表，并插入一条数据。

   ```SQL
   -- Create a table.
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
   
   -- Prepared data.
   mysql > INSERT INTO test_tbl4 (id, name, job1, job2) VALUES (0, 0, 1, 1);
   Query OK, 1 row affected (0.04 sec)
   {'label':'insert_6af28e77-7d2b-11ed-af6e-02424283676b', 'status':'VISIBLE', 'txnId':'152'}
   
   -- Query the table.
   mysql > SELECT * FROM test_tbl4 ORDER BY id;
   +------+------+------+------+
   | id   | name | job1 | job2 |
   +------+------+------+------+
   |    0 |    0 |    1 |    1 |
   +------+------+------+------+
   1 row in set (0.01 sec)
   ```

2. 准备CSV文件**my_data4.csv**以更新表`test_tbl4`。CSV文件包括`AUTO_INCREMENT`列的值，但不包括`job1`列的值。第一行的主键在表`test_tbl4`中已存在，而第二行的主键在表中不存在。

   ```Plaintext
   0,0,99
   1,1,99
   ```

3. 运行一个[Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)作业，并使用CSV文件来更新表`test_tbl4`。

   ```Bash
   curl --location-trusted -u <username>:<password> -H "label:1" \
       -H "column_separator:," \
       -H "partial_update:true" \
       -H "columns:id,name,job2" \
       -T my_data4.csv -XPUT \
       http://<fe_host>:<fe_http_port>/api/example_db/test_tbl4/_stream_load
   ```

4. 查询更新后的表。test_tbl4表中第一行数据已存在，job1列的值保持不变。第二行数据为新插入，由于未指定job1列的默认值，部分更新框架直接将该列的值设置为0。

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

### AUTO_INCREMENT列非主键

如果AUTO_INCREMENT列不是主键或主键的一部分，在流式加载作业中未提供自增ID时，将出现以下情况：

- 如果行已在表中存在，StarRocks不会更新自增ID。
- 如果行是新加载到表中的，StarRocks将生成新的自增ID。

此特性可用于构建字典表，以快速计算不同的STRING值。

1. 在example_db数据库中创建一个名为test_tbl5的表，指定job1列为AUTO_INCREMENT列，并向test_tbl5表中插入一条数据。

   ```SQL
   -- Create a table.
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
   
   -- Prepare data.
   mysql > INSERT INTO test_tbl5 VALUES (0, 0, -1, -1);
   Query OK, 1 row affected (0.04 sec)
   {'label':'insert_458d9487-80f6-11ed-ae56-aa528ccd0ebf', 'status':'VISIBLE', 'txnId':'94'}
   
   -- Query the table.
   mysql > SELECT * FROM test_tbl5 ORDER BY id;
   +------+------+------+------+
   | id   | name | job1 | job2 |
   +------+------+------+------+
   |    0 |    0 |   -1 |   -1 |
   +------+------+------+------+
   1 row in set (0.01 sec)
   ```

2. 准备一个CSV文件**my_data5.csv**来更新表`test_tbl5`。CSV文件不包含`AUTO_INCREMENT`列`job1`的值。第一行的主键在表中已存在，而第二和第三行的主键不存在。

   ```Plaintext
   0,0,99
   1,1,99
   2,2,99
   ```

3. 运行一个[流式加载](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)作业，将数据从CSV文件加载到`test_tbl5`表中。

   ```Bash
   curl --location-trusted -u <username>:<password> -H "label:2" \
       -H "column_separator:," \
       -H "partial_update:true" \
       -H "columns: id,name,job2" \
       -T my_data5.csv -XPUT \
       http://<fe_host>:<fe_http_port>/api/example_db/test_tbl5/_stream_load
   ```

4. 查询更新后的表。test_tbl5表中第一行数据已存在，AUTO_INCREMENT列job1保留其原始值。第二、三行数据为新插入，StarRocks为AUTO_INCREMENT列job1生成了新值。

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

- 创建带有AUTO_INCREMENT列的表时，必须设置'replicated_storage' = 'true'，以确保所有副本拥有相同的自增ID。
- 每个表只能有一个AUTO_INCREMENT列。
- AUTO_INCREMENT列的数据类型必须是BIGINT。
- AUTO_INCREMENT列必须为NOT NULL且没有默认值。
- 您可以从带有AUTO_INCREMENT列的主键表中删除数据。但是，如果AUTO_INCREMENT列不是主键或主键的一部分，在以下情况下删除数据时，需要注意以下限制：

  - 在DELETE操作期间，如果还有针对部分更新的加载作业，且仅包含UPSERT操作，如果UPSERT和DELETE操作同时命中同一数据行，并且UPSERT操作在DELETE操作之后执行，UPSERT操作可能不会生效。
  - 如果有一个针对部分更新的加载作业，包含对同一数据行的多个UPSERT和DELETE操作，如果某个UPSERT操作在DELETE操作之后执行，UPSERT操作可能不会生效。

- 不支持使用ALTER TABLE添加AUTO_INCREMENT属性。
- 从3.1版本开始，StarRocks的共享数据模式支持AUTO_INCREMENT属性。
- StarRocks不支持指定AUTO_INCREMENT列的起始值和步长。

## 关键字

AUTO_INCREMENT，自动增量
