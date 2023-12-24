---
displayed_sidebar: English
---

# 布隆过滤器索引

本主题描述了如何创建和修改布隆过滤器索引，以及它们的工作原理。

布隆过滤器索引是一种高效利用空间的数据结构，用于检测表的数据文件中是否可能存在被筛选的数据。如果布隆过滤器索引检测到某个数据文件中不包含需要筛选的数据，StarRocks 将跳过扫描该数据文件。布隆过滤器索引可在列（如 ID）的基数相对较高时缩短响应时间。

当查询命中排序键列时，StarRocks 会使用[前缀索引](../table_design/Sort_key.md)高效返回查询结果。但是，数据块的前缀索引条目长度不能超过 36 个字节。如果要提高非排序键且基数相对较高的列的查询性能，可以为该列创建布隆过滤器索引。

## 工作原理

例如，在给定表的 `table1` 上为 `column1` 创建了布隆过滤器索引，并运行了如 `Select xxx from table1 where column1 = something;` 的查询。那么当 StarRocks 扫描 `table1` 的数据文件时，会出现以下情况。

- 如果布隆过滤器索引检测到某个数据文件不包含需要筛选的数据，StarRocks 将跳过该数据文件以提高查询性能。
- 如果布隆过滤器索引检测到某个数据文件可能包含需要筛选的数据，StarRocks 将读取该数据文件以检查数据是否存在。请注意，布隆过滤器可以确定某个值是否不存在，但无法确定某个值是否存在，只能说它可能存在。使用布隆过滤器索引确定值是否存在可能会产生误报，这意味着布隆过滤器索引检测到数据文件包含要筛选的数据，但数据文件实际上并不包含这些数据。

## 使用说明

- 您可以为“重复键”或“主键”表的所有列创建布隆过滤器索引。对于聚合表或唯一键表，只能为键列创建布隆过滤器索引。
- TINYINT、FLOAT、DOUBLE 和 DECIMAL 列不支持创建布隆过滤器索引。
- 布隆过滤器索引只能提高包含 `in` 和 `=` 运算符的查询的性能，例如 `Select xxx from table where x in {}` 和 `Select xxx from table where column = xxx`。
- 您可以通过查看查询的配置文件中的`BloomFilterFilterRows`字段来检查查询是否使用了布隆过滤器索引。

## 创建布隆过滤器索引

在创建表时，可以通过在 `PROPERTIES` 中指定 `bloom_filter_columns` 参数来为列创建布隆过滤器索引。例如，在 `table1` 中为 `k1` 和 `k2` 列创建布隆过滤器索引。

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

通过指定这些列名，可以一次为多个列创建布隆过滤器索引。请注意，需要用逗号（`,`）分隔这些列名。有关 CREATE TABLE 语句的其他参数说明，请参见 [CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。

## 显示布隆过滤器索引

例如，以下语句显示了 `table1` 的布隆过滤器索引。有关输出说明，请参见 [SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md)。

```SQL
SHOW CREATE TABLE table1;
```

## 修改布隆过滤器索引

您可以使用 [ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md) 语句添加、减少和删除布隆过滤器索引。

- 以下语句在列 `v1` 上添加了布隆过滤器索引。

    ```SQL
    ALTER TABLE table1 SET ("bloom_filter_columns" = "k1,k2,v1");
    ```

- 以下语句减少了列 `k2` 上的布隆过滤器索引。
  
    ```SQL
    ALTER TABLE table1 SET ("bloom_filter_columns" = "k1");
    ```

- 以下语句删除了 `table1` 的所有布隆过滤器索引。

    ```SQL
    ALTER TABLE table1 SET ("bloom_filter_columns" = "");
    ```

> 注意：更改索引是异步操作。您可以通过执行 [SHOW ALTER TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_ALTER.md) 来查看此操作的进度。每次只能对一个表运行一个 alter index 任务。
