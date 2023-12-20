---
displayed_sidebar: English
---

# 位图索引

本主题旨在介绍如何创建和管理位图索引，以及其使用场景。

位图索引是一种特殊的数据库索引，它使用位图，而位图是由位组成的数组。位的值只有两种可能：0 或 1。位图中的每一位都对应表中的一条记录。每一位的值取决于其对应记录的值。

位图索引能够帮助提升特定列上的查询性能。如果查询涉及排序键列，StarRocks 会利用[前缀索引](../table_design/Sort_key.md)高效地返回查询结果。然而，数据块的前缀索引条目长度不能超过 36 字节。如果您想提升非排序键列的查询性能，可以为该列创建位图索引。

## 优势

位图索引在以下几个方面能带来好处：

- 当列的基数较低时，如 ENUM 类型的列，可以减少响应时间。如果某列的不同值数量较多，我们推荐使用布隆过滤器索引来提高查询速度。更多信息请参考[布隆过滤器索引](../using_starrocks/Bloomfilter_index.md)。
- 与其他索引技术相比，位图索引占用的存储空间更少。位图索引通常只占表中被索引数据大小的一小部分。
- 可以组合多个位图索引来同时查询多个列。更多信息请参考[查询多个列](#query-multiple-columns)。

## 使用说明

- 您可以为可以通过等于（=）或 [NOT] IN 运算符筛选的列创建位图索引。
- 您可以为重复键表或唯一键表的所有列创建位图索引。对于聚合表或主键表，您只能为键列创建位图索引。
- FLOAT、DOUBLE、BOOLEAN、DECIMAL 类型的列不支持创建位图索引。
- 您可以通过查看查询的 Profile 中的 BitmapIndexFilterRows 字段来检查查询是否使用了位图索引。

## 创建位图索引

为列创建位图索引有两种方法：

- 在创建表时为列创建位图索引。例如：

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

  以下表格描述了与位图索引相关的参数。

  |参数|必填|说明|
|---|---|---|
  |index_name|是|位图索引的名称。命名规则如下：名称可以包含字母、数字（0-9）和下划线（_）。必须以字母开头。名称长度不能超过 64 个字符。位图索引的名称在表中必须唯一。|
  |column_name|Yes|创建位图索引的列的名称。您可以指定多个列名来一次为多个列创建位图索引。用逗号 (,) 分隔多列。|
  |COMMENT|否|位图索引的注释。|

  您可以通过指定多个 `INDEX index_name (column_name [, ...]) [USING BITMAP] [COMMENT '注释']` 命令来同时为多个列创建位图索引，这些命令之间需要用逗号（`,`）分隔。关于 CREATE TABLE 语句的其他参数描述，请参见 [CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。

- 使用 CREATE INDEX 语句为表中的列创建位图索引。有关参数描述和示例，请参见 [CREATE INDEX](../sql-reference/sql-statements/data-definition/CREATE_INDEX.md)。

  ```SQL
  CREATE INDEX index_name ON table_name (column_name) [USING BITMAP] [COMMENT ''];
  ```

## 查看位图索引

您可以使用 SHOW INDEX 语句查看表中创建的所有位图索引。有关参数描述和示例，请参见 [SHOW INDEX](../sql-reference/sql-statements/Administration/SHOW_INDEX.md)。

```SQL
SHOW { INDEX[ES] | KEY[S] } FROM [db_name.]table_name [FROM db_name];
```

> **注意**
> 创建索引是一个**异步**过程。因此，您只能看到那些已完成**创建**过程的索引。

## 删除位图索引

您可以使用 DROP INDEX 语句从表中删除位图索引。有关参数描述和示例，请参见 [DROP INDEX](../sql-reference/sql-statements/data-definition/DROP_INDEX.md)。

```SQL
DROP INDEX index_name ON [db_name.]table_name;
```

## 使用案例

例如，下表 employee 展示了某公司员工的部分信息。

|ID|性别|职位|收入水平|
|---|---|---|---|
|01|女性|开发者|level_1|
|02|女|分析师|level_2|
|03|女|推销员|level_1|
|04|男|会计师|level_3|

### 查询单个列

例如，如果您希望提高性别（Gender）列的查询性能，可以使用以下语句为该列创建位图索引。

```SQL
CREATE INDEX index1 ON employee (Gender) USING BITMAP COMMENT 'index1';
```

执行上述语句后，将生成如下所示的位图索引。

![figure](../assets/3.6.1-2.png)

1. 构建字典：StarRocks 为性别（Gender）列构建字典，并将“女”和“男”映射为 INT 类型的编码值：0 和 1。
2. 生成位图：StarRocks 根据编码值为“女”和“男”生成位图。“女”的位图是 1110，因为前三行都是女性。“男”的位图是 0001，因为只有第四行是男性。

如果您想查询公司中的男性员工，可以执行以下查询。

```SQL
SELECT xxx FROM employee WHERE Gender = male;
```

查询执行后，StarRocks 查找字典以获取“男”的编码值，即 1，然后获取男性的位图，即 0001。这意味着只有第四行符合查询条件。接着，StarRocks 将跳过前三行，只读取第四行。

### 查询多个列

例如，如果您希望提高性别（Gender）和收入水平（Income_level）两列的查询性能，可以使用以下语句为这两列创建位图索引。

- 性别（Gender）

  ```SQL
  CREATE INDEX index1 ON employee (Gender) USING BITMAP COMMENT 'index1';
  ```

- 收入水平（Income_level）

  ```SQL
  CREATE INDEX index2 ON employee (Income_level) USING BITMAP COMMENT 'index2';
  ```

执行以上两条语句后，将如下图所示生成位图索引。

![figure](../assets/3.6.1-3.png)

StarRocks 分别为性别（Gender）和收入水平（Income_level）列构建字典，然后为这两列的不同值生成位图。

- 性别：女性的位图是 1110，男性的位图是 0001。
- 收入水平：level_1 的位图是 1010，level_2 的位图是 0100，level_3 的位图是 0001。

如果您想查询收入水平为 level_1 的女性员工，可以执行以下查询。

```SQL
 SELECT xxx FROM employee 
 WHERE Gender = female AND Income_level = level_1;
```

查询执行后，StarRocks 同时查找性别（Gender）和收入水平（Income_level）的字典，得到以下信息：

- 女性的编码值是 0，女性的位图是 1110。
- level_1 的编码值是 0，level_1 的位图是 1010。

StarRocks 通过 AND 运算符执行按位逻辑运算 1110 & 1010，得到结果 1010。根据这个结果，StarRocks 只读取第一行和第三行。
