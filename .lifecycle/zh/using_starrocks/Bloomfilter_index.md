---
displayed_sidebar: English
---

# 布隆过滤器索引

本主题将介绍如何创建和修改布隆过滤器索引，以及它们的工作原理。

布隆过滤器索引是一种空间高效的数据结构，用于检测表中数据文件里可能存在的过滤数据。如果布隆过滤器索引判断出某数据文件中不包含待过滤的数据，StarRocks 将跳过扫描该数据文件。对于基数相对较高的列（如ID）来说，布隆过滤器索引能够缩短响应时间。

如果查询命中了排序键列，StarRocks将利用[前缀索引](../table_design/Sort_key.md)高效返回查询结果。但是，数据块的前缀索引条目长度不能超过36字节。如果您希望提升非排序键且基数较高的列的查询性能，您可以为该列创建布隆过滤器索引。

## 工作原理

例如，您在某表 table1 的 column1 上创建了一个布隆过滤器索引，并执行了类似 Select xxx from table1 where column1 = something; 的查询。当StarRocks 扫描 table1 的数据文件时，会发生以下情况：

- 如果布隆过滤器索引判断出某数据文件不包含待过滤的数据，StarRocks 会跳过该数据文件，以提升查询性能。
- 如果布隆过滤器索引判断出某数据文件可能包含待过滤的数据，StarRocks 会读取该数据文件以确认数据是否存在。注意，布隆过滤器可以确切地告诉您某个值不存在，但不能确切地告诉您某个值一定存在，只能表明它可能存在。使用布隆过滤器索引来判断值是否存在可能会产生误报，即布隆过滤器索引显示数据文件可能包含待过滤的数据，但实际上该数据文件并未包含该数据。

## 使用须知

- 您可以为重复键表或主键表的所有列创建布隆过滤器索引。对于聚合表或唯一键表，您只能为关键列创建布隆过滤器索引。
- TINYINT、FLOAT、DOUBLE 和 DECIMAL 类型的列不支持创建布隆过滤器索引。
- 布隆过滤器索引只能提升包含 in 和 = 运算符的查询性能，例如 Select xxx from table where x in {} 和 Select xxx from table where column = xxx。
- 您可以通过查看查询的执行计划中的 BloomFilterFilterRows 字段来检查查询是否利用了布隆过滤器索引。

## 创建布隆过滤器索引

您可以在创建表时，通过在 PROPERTIES 中指定 bloom_filter_columns 参数为列创建布隆过滤器索引。例如，为 table1 中的 k1 和 k2 列创建布隆过滤器索引。

```SQL
CREATE TABLE table1
(
    k1 BIGINT,
    k2 LARGEINT,
    v1 VARCHAR(2048) REPLACE,
    v2 SMALLINT DEFAULT "10"
)
ENGINE = olap
PRIMARY KEY(k1, k2)
DISTRIBUTED BY HASH (k1, k2)
PROPERTIES("bloom_filter_columns" = "k1,k2");
```

您可以同时为多个列创建布隆过滤器索引，只需在指定列名时用逗号（`,`）分隔。关于 CREATE TABLE 语句的其他参数说明，请参阅 [CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。

## 显示布隆过滤器索引

例如，以下语句用于展示`table1`的布隆过滤器索引。输出描述请参见[SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md)。

```SQL
SHOW CREATE TABLE table1;
```

## 修改布隆过滤器索引

您可以使用[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)语句来增加、减少或删除布隆过滤器索引。

- 以下语句用于在 v1 列上增加一个布隆过滤器索引。

  ```SQL
  ALTER TABLE table1 SET ("bloom_filter_columns" = "k1,k2,v1");
  ```

- 以下语句用于减少 k2 列上的布隆过滤器索引。

  ```SQL
  ALTER TABLE table1 SET ("bloom_filter_columns" = "k1");
  ```

- 以下语句用于删除 table1 的所有布隆过滤器索引。

  ```SQL
  ALTER TABLE table1 SET ("bloom_filter_columns" = "");
  ```

> 注意：修改索引是一个异步操作。您可以通过执行 [SHOW ALTER TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_ALTER.md) 来查看此操作的进度。在一张表上，每次只能执行一个索引修改任务。
