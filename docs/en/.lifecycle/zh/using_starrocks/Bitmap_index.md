---
displayed_sidebar: English
---

# 位图索引

本主题介绍如何创建和管理位图索引以及使用案例。

位图索引是一种特殊的数据库索引，它使用位图，即位数组。每个位都只能是0或1。位图中的每一位对应表中的一行，其值取决于相应行的值。

位图索引可帮助提高对特定列的查询性能。当查询命中排序键列时，StarRocks会使用[前缀索引](../table_design/Sort_key.md)高效返回查询结果。但是，数据块的前缀索引条目长度不能超过36个字节。如果要提高非排序键列的查询性能，可以为该列创建位图索引。

## 好处

您可以从位图索引中获得以下好处：

- 当列的基数较小时，例如ENUM类型的列，可缩短响应时间。如果列中的不同值数量相对较多，建议您使用布隆过滤器索引来提高查询速度。有关详细信息，请参见[布隆过滤器索引](../using_starrocks/Bloomfilter_index.md)。
- 与其他索引技术相比，位图索引使用的存储空间更少。位图索引通常只占表中索引数据大小的一小部分。
- 可以将多个位图索引组合在一起，以便对多个列进行查询。有关详细信息，请参见[查询多个列](#query-multiple-columns)。

## 使用说明

- 可以为可以使用等于（=）或[NOT] IN运算符进行筛选的列创建位图索引。
- 可以为“重复键”表或“唯一键”表的所有列创建位图索引。对于聚合表或主键表，只能为键列创建位图索引。
- FLOAT、DOUBLE、BOOLEAN和DECIMAL类型的列不支持创建位图索引。
- 您可以通过查看查询配置文件的字段`BitmapIndexFilterRows`来检查查询是否使用位图索引。

## 创建位图索引

有两种方法可以为列创建位图索引。

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

    下表描述了与位图索引相关的参数。

    | **参数** | **必填** | **描述**                                              |
    | ------------- | ------------ | ------------------------------------------------------------ |
    | index_name    | 是          | 位图索引的名称。 命名约定如下：<ul><li>名称可以包含字母、数字（0-9）、下划线（_）。必须以字母开头。</li><li>名称长度不能超过64个字符。</li></ul>位图索引的名称在表中必须是唯一的。                              |
    | column_name   | 是          | 要创建位图索引的列的名称。您可以一次指定多个列名，以便为多个列创建位图索引。用逗号（,）分隔多个列。  |
    | COMMENT       | 否          | 位图索引的注释。                             |

  通过指定多个`INDEX index_name (column_name [, ...]) [USING BITMAP] [COMMENT '']`命令，可以一次为多个列创建位图索引。这些命令需要用逗号（,）分隔。有关CREATE TABLE语句的其他参数说明，请参见[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。

- 使用CREATE INDEX语句为表的列创建位图索引。有关参数说明和示例，请参见[CREATE INDEX](../sql-reference/sql-statements/data-definition/CREATE_INDEX.md)。

    ```SQL
    CREATE INDEX index_name ON table_name (column_name) [USING BITMAP] [COMMENT ''];
    ```

## 显示位图索引

您可以使用SHOW INDEX语句查看表中创建的所有位图索引。有关参数说明和示例，请参见[SHOW INDEX](../sql-reference/sql-statements/Administration/SHOW_INDEX.md)。

```SQL
SHOW { INDEX[ES] | KEY[S] } FROM [db_name.]table_name [FROM db_name];
```

> **注意**
>
> 创建索引是一个异步过程。因此，您只能看到已完成创建过程的索引。

## 删除位图索引

可以使用DROP INDEX语句从表中删除位图索引。有关参数说明和示例，请参见[DROP INDEX](../sql-reference/sql-statements/data-definition/DROP_INDEX.md)。

```SQL
DROP INDEX index_name ON [db_name.]table_name;
```

## 使用案例

例如，下表`employee`显示了公司员工信息的一部分。

| **ID** | **Gender** | **Position** | **Income_level** |
| ------ | ---------- | ------------ | ---------------- |
| 01     | 女       | 开发员       | level_1          |
| 02     | 女       | 分析员       | level_2          |
| 03     | 女       | 推销员       | level_1          |
| 04     | 男       | 会计         | level_3          |

### 查询单个列

例如，如果要提高对`Gender`列的查询性能，可以使用以下语句为该列创建位图索引。

```SQL
CREATE INDEX index1 ON employee (Gender) USING BITMAP COMMENT 'index1';
```

执行上述语句后，将生成如下图所示的位图索引。

![图例](../assets/3.6.1-2.png)

1. 构建字典：StarRocks为`Gender`列构建字典，并将`female`和`male`映射为INT类型的编码值：`0`和`1`。
2. 生成位图：StarRocks根据编码值为`female`和`male`生成位图。`female`的位图为`1110`，因为`female`显示在前三行。`male`的位图为`0001`，因为`male`只显示在第四行。

如果要查找公司中的男性员工，可以发送以下查询。

```SQL
SELECT xxx FROM employee WHERE Gender = male;
```

查询发送后，StarRocks会查找字典，获取`male`的编码值，即`1`，然后获取`male`的位图，即`0001`。这意味着只有第四行符合查询条件。然后StarRocks将跳过前三行，只读取第四行。

### 查询多个列

例如，如果要提高对`Gender`和`Income_level`列的查询性能，可以使用以下语句为这两列创建位图索引。

- `Gender`

    ```SQL
    CREATE INDEX index1 ON employee (Gender) USING BITMAP COMMENT 'index1';
    ```

- `Income_level`

    ```SQL
    CREATE INDEX index2 ON employee (Income_level) USING BITMAP COMMENT 'index2';
    ```

执行上述两条语句后，将生成如下图所示的位图索引。

![图例](../assets/3.6.1-3.png)

StarRocks分别为`Gender`和`Income_level`列构建字典，然后为这两列中的不同值生成位图。

- `Gender`：`female`的位图为`1110`，`male`的位图为`0001`。
- `Income_level`：`level_1`的位图为`1010`，`level_2`的位图为`0100`，`level_3`的位图为`0001`。

如果要查找工资在`level_1`的女性员工，可以发送以下查询。

```SQL
SELECT xxx FROM employee 
WHERE Gender = female AND Income_level = level_1;
```

查询发送后，StarRocks同时查找`Gender`和`Income_level`的字典，获取以下信息：

- `female`的编码值为`0`，`female`的位图为`1110`。
- `level_1`的编码值为`0`，`level_1`的位图为`1010`。

StarRocks根据`AND`运算符进行按位逻辑运算`1110 & 1010`，得到结果`1010`。根据结果，StarRocks只读取第一行和第三行。
