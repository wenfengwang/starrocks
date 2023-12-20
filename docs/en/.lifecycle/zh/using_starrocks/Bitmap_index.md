---
displayed_sidebar: English
---

# 位图索引

本主题介绍如何创建和管理位图索引以及使用案例。

位图索引是一种特殊的数据库索引，它使用位图，位图是一个位数组。位总是有两个值之一：0 或 1。位图中的每个位对应表中的一行。每个位的值取决于相应行的值。

位图索引可以帮助提高特定列的查询性能。如果查询命中排序键列，StarRocks 会使用[前缀索引](../table_design/Sort_key.md)高效返回查询结果。然而，数据块的前缀索引条目长度不能超过 36 字节。如果您想提高非排序键列的查询性能，您可以为该列创建位图索引。

## 优势

您可以从以下几个方面获得位图索引的好处：

- 当列的基数较低时，如 ENUM 类型的列，可以减少响应时间。如果列中不同值的数量较多，我们建议您使用[布隆过滤器索引](../using_starrocks/Bloomfilter_index.md)来提高查询速度。有关更多信息，请参阅布隆过滤器索引。
- 与其他索引技术相比，占用更少的存储空间。位图索引通常只占用表中索引数据大小的一小部分。
- 结合多个位图索引来执行多列的查询。有关更多信息，请参阅[查询多列](#query-multiple-columns)。

## 使用说明

- 您可以为可以使用等于（`=`）或 [NOT] IN 运算符过滤的列创建位图索引。
- 您可以为 Duplicate Key 表或 Unique Key 表的所有列创建位图索引。对于 Aggregate 表或 Primary Key 表，您只能为键列创建位图索引。
- FLOAT、DOUBLE、BOOLEAN 和 DECIMAL 类型的列不支持创建位图索引。
- 您可以通过查看查询的 `BitmapIndexFilterRows` 字段来检查查询是否使用了位图索引。

## 创建位图索引

有两种方法可以为列创建位图索引。

- 在创建表时为列创建位图索引。示例：

  ```SQL
  CREATE TABLE d0.table_hash
  (
      k1 TINYINT,
      k2 DECIMAL(10, 2) DEFAULT "10.5",
      v1 CHAR(10) REPLACE,
      v2 INT SUM,
      INDEX index_name (column_name [, ...]) [USING BITMAP] [COMMENT '']
  )
  ENGINE = olap
  AGGREGATE KEY(k1, k2)
  DISTRIBUTED BY HASH(k1)
  PROPERTIES ("storage_type" = "column");
  ```

  下表描述了与位图索引相关的参数。

  |参数|必需|描述|
|---|---|---|
  |index_name|是|位图索引的名称。命名规则如下：<ul><li>名称可以包含字母、数字（0-9）和下划线（_）。必须以字母开头。</li><li>名称长度不能超过 64 个字符。</li></ul>位图索引的名称在表中必须是唯一的。|
  |column_name|是|创建位图索引的列的名称。您可以指定多个列名，一次为多个列创建位图索引。用逗号（`,`）分隔多个列。|
  |COMMENT|否|位图索引的注释。|

  您可以通过指定多个 `INDEX index_name (column_name [, ...]) [USING BITMAP] [COMMENT '']` 命令，一次为多个列创建位图索引。这些命令需要用逗号（`,`）分隔。有关 CREATE TABLE 语句的其他参数描述，请参见 [CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。

- 使用 CREATE INDEX 语句为表中的列创建位图索引。有关参数描述和示例，请参见 [CREATE INDEX](../sql-reference/sql-statements/data-definition/CREATE_INDEX.md)。

  ```SQL
  CREATE INDEX index_name ON table_name (column_name) [USING BITMAP] [COMMENT ''];
  ```

## 显示位图索引

您可以使用 SHOW INDEX 语句查看表中创建的所有位图索引。有关参数描述和示例，请参见 [SHOW INDEX](../sql-reference/sql-statements/Administration/SHOW_INDEX.md)。

```SQL
SHOW { INDEX[ES] | KEY[S] } FROM [db_name.]table_name [FROM db_name];
```

> **注意**
> 创建索引是一个异步过程。因此，您只能看到已完成创建过程的索引。

## 删除位图索引

您可以使用 DROP INDEX 语句从表中删除位图索引。有关参数描述和示例，请参见 [DROP INDEX](../sql-reference/sql-statements/data-definition/DROP_INDEX.md)。

```SQL
DROP INDEX index_name ON [db_name.]table_name;
```

## 使用案例

例如，下表 `employee` 显示了公司部分员工的信息。

|ID|性别|职位|收入等级|
|---|---|---|---|
|01|女性|开发者|level_1|
|02|女性|分析师|level_2|
|03|女性|销售员|level_1|
|04|男性|会计|level_3|

### 查询单列

例如，如果您想提高 `Gender` 列的查询性能，可以使用以下语句为该列创建位图索引。

```SQL
CREATE INDEX index1 ON employee (Gender) USING BITMAP COMMENT 'index1';
```

执行上述语句后，生成的位图索引如下图所示。

![figure](../assets/3.6.1-2.png)

1. 构建字典：StarRocks 为 `Gender` 列构建字典，并将 `female` 和 `male` 映射到 INT 类型的编码值：`0` 和 `1`。
2. 生成位图：StarRocks 根据编码值为 `female` 和 `male` 生成位图。`female` 的位图是 `1110`，因为 `female` 出现在前三行。`male` 的位图是 `0001`，因为 `male` 仅出现在第四行。

如果您想找出公司中的男性员工，可以发送如下查询。

```SQL
SELECT xxx FROM employee WHERE Gender = male;
```

查询发送后，StarRocks 查找字典以获取 `male` 的编码值，即 `1`，然后获取 `male` 的位图，即 `0001`。这意味着只有第四行符合查询条件。然后 StarRocks 将跳过前三行，只读取第四行。

### 查询多列

例如，如果您想提高 `Gender` 和 `Income_level` 列的查询性能，可以使用以下语句为这两列创建位图索引。

- `Gender`

  ```SQL
  CREATE INDEX index1 ON employee (Gender) USING BITMAP COMMENT 'index1';
  ```

- `Income_level`

  ```SQL
  CREATE INDEX index2 ON employee (Income_level) USING BITMAP COMMENT 'index2';
  ```

执行上述两条语句后，生成的位图索引如下图所示。

![figure](../assets/3.6.1-3.png)

StarRocks 分别为 `Gender` 和 `Income_level` 列构建字典，然后为这两列中的不同值生成位图。

- `Gender`：`female` 的位图是 `1110`，`male` 的位图是 `0001`。
- `Income_level`：`level_1` 的位图是 `1010`，`level_2` 的位图是 `0100`，`level_3` 的位图是 `0001`。

如果您想找出工资等级为 `level_1` 的女性员工，可以发送如下查询。

```SQL
 SELECT xxx FROM employee 
 WHERE Gender = female AND Income_level = level_1;
```

查询发送后，StarRocks 同时搜索 `Gender` 和 `Income_level` 的字典，得到以下信息：

- `female` 的编码值为 `0`，位图为 `1110`。
- `level_1` 的编码值为 `0`，位图为 `1010`。

StarRocks 根据 `AND` 运算符执行按位逻辑运算 `1110 & 1010`，得到结果 `1010`。根据结果，StarRocks 只读取第一行和第三行。