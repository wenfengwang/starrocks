---
displayed_sidebar: "中文"
---

# 位图索引

本主题介绍如何创建和管理位图索引，以及使用案例。

位图索引是一种特殊的数据库索引，它使用位图，而位图则是位数组。位始终处于两个值中的一个：0和1。位图中的每个位都对应表中的一行。位的值取决于相应行的值。

位图索引可以帮助提高对给定列的查询性能。如果查询命中了排序键列，StarRocks将通过使用[前缀索引](../table_design/Sort_key.md)高效返回查询结果。然而，数据块的前缀索引条目长度不能超过36个字节。如果您希望提高非排序键列的查询性能，则可以为该列创建位图索引。

## 好处

您可以从位图索引中获得以下各方面的好处：

- 当列的基数较低时，例如 ENUM 类型的列时，可降低响应时间。若列中的不同值的数量相对较高，则建议您使用布隆过滤器索引来提高查询速度。有关更多信息，请参见[布隆过滤器索引](../using_starrocks/Bloomfilter_index.md)。
- 相较于其他索引技术，位图索引占用的存储空间更少。位图索引通常只占表中索引数据大小的一小部分。
- 可将多个位图索引组合在一起，以查询多个列。有关更多信息，请参见[查询多个列](#query-multiple-columns)。

## 使用注意事项

- 您可以为可通过等于 (`=`) 或 [NOT] IN 运算符进行过滤的列创建位图索引。
- 您可以为具有重复键表或唯一键表的所有列创建位图索引。对于聚合表或主键表，您只能为键列创建位图索引。
- FLOAT、DOUBLE、BOOLEAN 和 DECIMAL 类型的列不支持创建位图索引。
- 您可以通过查看查询的概要文件的 `BitmapIndexFilterRows` 字段，来检查查询是否使用了位图索引。

## 创建位图索引

有两种方法可为列创建位图索引。

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

    | **参数**     | **是否必需** | **描述**                                                     |
    | ------------ | ------------ | ------------------------------------------------------------ |
    | index_name   | 是           | 位图索引的名称。 命名约定如下：<ul><li>名称可以包含字母、数字（0-9）和下划线（_）。必须以字母开头。</li><li>名称长度不能超过64个字符。</li></ul>位图索引的名称在表中必须是唯一的。               |
    | column_name  | 是           | 创建位图索引的列的名称。您可以指定多个列名，同时为多个列创建位图索引。请使用逗号（`,`）将多个列分隔开。  |
    | COMMENT      | 否           | 位图索引的注释。                                             |

    您可以通过指定多个 `INDEX index_name (column_name [, ...]) [USING BITMAP] [COMMENT '']` 命令，一次性为多个列创建位图索引。这些命令需用逗号（`,`）分隔。有关 CREATE TABLE 语句的其他参数说明，请参见[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。

- 使用 CREATE INDEX 语句为表的列创建位图索引。有关参数说明和示例，请参见[CREATE INDEX](../sql-reference/sql-statements/data-definition/CREATE_INDEX.md)。

    ```SQL
    CREATE INDEX index_name ON table_name (column_name) [USING BITMAP] [COMMENT ''];
    ```

## 显示位图索引

您可以使用 SHOW INDEX 语句查看表中创建的所有位图索引。有关参数说明和示例，请参见[SHOW INDEX](../sql-reference/sql-statements/Administration/SHOW_INDEX.md)。

```SQL
SHOW { INDEX[ES] | KEY[S] } FROM [db_name.]table_name [FROM db_name];
```

> **说明**
>
> 索引的创建是一个异步过程。因此，只能查看已完成创建过程的索引。


## 删除位图索引

您可以使用 DROP INDEX 语句从表中删除位图索引。有关参数说明和示例，请参见[DROP INDEX](../sql-reference/sql-statements/data-definition/DROP_INDEX.md)。

```SQL
DROP INDEX index_name ON [db_name.]table_name;
```

## 使用案例

例如，以下表 `employee` 显示了公司员工信息的部分内容。

| **ID** | **Gender** | **Position** | **Income_level** |
| ------ | ---------- | ------------ | ---------------- |
| 01     | female     | Developer    | level_1          |
| 02     | female     | Analyst      | level_2          |
| 03     | female     | Salesman     | level_1          |
| 04     | male       | Accountant   | level_3          |

### 查询单列

例如，如果您希望改善对 `Gender` 列的查询性能，可以使用以下语句为该列创建位图索引。

```SQL
CREATE INDEX index1 ON employee (Gender) USING BITMAP COMMENT 'index1';
```

执行上述语句后，将生成位图索引，如下图所示。

![figure](../assets/3.6.1-2.png)

1. 构建字典：StarRocks为 `Gender` 列构建字典，并将 `female` 和 `male` 映射到 INT 类型的编码值：`0` 和 `1`。
2. 生成位图：StarRocks根据编码值为 `female` 和 `male` 生成位图。`female` 的位图为 `1110`，因为 `female` 在前三行中显示。`male` 的位图为 `0001`，因为 `male` 只在第四行显示。

如果您想了解公司中的男性员工，可以发送以下查询。

```SQL
SELECT xxx FROM employee WHERE Gender = male;
```

查询发送后，StarRocks搜索字典以获得 `male` 的编码值，即 `1`，然后获取 `male` 的位图，即 `0001`。这意味着只有第四行符合查询条件。然后，StarRocks将跳过前三行，仅读取第四行。

### 查询多列

例如，如果您希望改善 `Gender` 和 `Income_level` 列的查询性能，可以使用以下语句为这两列创建位图索引。

- `Gender`

    ```SQL
    CREATE INDEX index1 ON employee (Gender) USING BITMAP COMMENT 'index1';
    ```

- `Income_level`

    ```SQL
    CREATE INDEX index2 ON employee (Income_level) USING BITMAP COMMENT 'index2';
    ```

执行上述两个语句后，将生成位图索引，如下图所示。

![figure](../assets/3.6.1-3.png)

StarRocks分别为 `Gender` 和 `Income_level` 列构建字典，然后为这两列的不同值生成位图。

- `Gender`：`female` 的位图为 `1110`，`male` 的位图为 `0001`。
- `Producer`：`level_1` 的位图为 `1010`，`level_2` 的位图为 `0100`，`level_3` 的位图为 `0001`。

如果您想找出薪水在 `level_1` 的女性员工，可以发送以下查询。

```SQL
 SELECT xxx FROM employee 
 WHERE Gender = female AND Income_level = level_1;
```

查询发送后，StarRocks同时搜索 `Gender` 和 `Income_level` 的字典以获取以下信息：

- `female` 的编码值为 `0`，位图为 `1110`。
- `level_1` 的编码值为 `0`，位图为 `1010`。

StarRocks根据 `AND` 操作符执行位逻辑运算 `1110 & 1010`，得到结果 `1010`。根据结果，StarRocks仅读取第一行和第三行。