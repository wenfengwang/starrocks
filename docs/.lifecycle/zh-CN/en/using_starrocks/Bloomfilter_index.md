---
displayed_sidebar: "Chinese"
---

# 布隆过滤器索引

本主题描述了如何创建和修改布隆过滤器索引，以及它们的工作原理。

布隆过滤器索引是一种用于检测表的数据文件中是否可能存在筛选数据的空间有效的数据结构。如果布隆过滤器索引检测到要筛选的数据不在某个数据文件中，StarRocks会跳过扫描该数据文件。布隆过滤器索引可以在列（如ID）的基数相对较高时减少响应时间。

如果查询命中了排序键列，StarRocks可以通过使用[前缀索引](../table_design/Sort_key.md)有效地返回查询结果。但是，数据块的前缀索引条目长度不能超过36个字节。如果您想要改善在未用作排序键且具有较高基数的列上的查询性能，可以为该列创建布隆过滤器索引。

## 工作原理

例如，您在给定表`table1`的列`column1`上创建了一个布隆过滤器索引，并运行了如`Select xxx from table1 where column1 = something;`的查询。那么当StarRocks扫描`table1`的数据文件时，会发生以下情况。

- 如果布隆过滤器索引检测到数据文件不包含要筛选的数据，则StarRocks会跳过该数据文件以提高查询性能。
- 如果布隆过滤器索引检测到数据文件可能包含要筛选的数据，则StarRocks会读取数据文件以检查数据是否存在。请注意，布隆过滤器可以确定一个值是否不存在，但不能确定一个值是否存在，只能确定一个值可能存在。使用布隆过滤器索引来确定一个值是否存在可能会产生误报阳性，这意味着布隆过滤器索引检测到数据文件包含要筛选的数据，但数据文件实际上不包含这些数据。

## 使用注意事项

- 您可以为重复键或主键表的所有列创建布隆过滤器索引。对于聚合表或唯一键表，您只能为键列创建布隆过滤器索引。
- TINYINT、FLOAT、DOUBLE和DECIMAL列不支持创建布隆过滤器索引。
- 布隆过滤器索引只能改善包含`in`和`=`操作符的查询性能，例如`Select xxx from table where x in {}`和`Select xxx from table where column = xxx`。
- 您可以通过查看查询概要的`BloomFilterFilterRows`字段来检查查询是否使用了布隆过滤器索引。

## 创建布隆过滤器索引

您可以通过在`PROPERTIES`中指定`bloom_filter_columns`参数在创建表时为列创建布隆过滤器索引。例如，在`table1`中为`k1`和`k2`列创建布隆过滤器索引。

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

您可以通过指定这些列名一次为多列创建布隆过滤器索引。请注意，您需要用逗号（`,`）分隔这些列名。有关`CREATE TABLE`语句的其他参数描述，请参见[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。

## 显示布隆过滤器索引

例如，以下语句显示了`table1`的布隆过滤器索引。有关输出描述，请参见[SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md)。

```SQL
SHOW CREATE TABLE table1;
```

## 修改布隆过滤器索引

您可以使用[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)语句添加、减少和删除布隆过滤器索引。

- 以下语句在`v1`列上添加了一个布隆过滤器索引。

    ```SQL
    ALTER TABLE table1 SET ("bloom_filter_columns" = "k1,k2,v1");
    ```

- 以下语句减少了`k2`列的布隆过滤器索引。
  
    ```SQL
    ALTER TABLE table1 SET ("bloom_filter_columns" = "k1");
    ```

- 以下语句删除了`table1`的所有布隆过滤器索引。

    ```SQL
    ALTER TABLE table1 SET ("bloom_filter_columns" = "");
    ```

> 注意：修改索引是一项异步操作。您可以通过执行[SHOW ALTER TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_ALTER.md)来查看此操作的进度。您一次只能在一个表上执行一个修改索引任务。