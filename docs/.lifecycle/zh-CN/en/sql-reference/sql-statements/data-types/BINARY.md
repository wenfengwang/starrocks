---
displayed_sidebar: "中文"
---

# BINARY/VARBINARY

## 描述

BINARY(M)

VARBINARY(M)

自v3.0起，StarRocks支持BINARY/VARBINARY数据类型，用于存储二进制数据。最大支持长度与VARCHAR相同（1~1048576）。单位为字节。如果未指定`M`，默认使用1048576。二进制数据类型包含字节字符串，而字符数据类型包含字符字符串。

BINARY是VARBINARY的别名。使用方法与VARBINARY相同。

## 限制和使用说明

- VARBINARY列支持在重复键、主键和唯一键表中。不支持在聚合表中。

- VARBINARY列不能用作重复键、主键和唯一键表的分区键、分桶键或维度列。不能在ORDER BY、GROUP BY和JOIN子句中使用。

- BINARY(M)/VARBINARY(M)在不对齐长度的情况下不会进行右填充。

## 示例

### 创建VARBINARY类型的列

在创建表时，使用关键字`VARBINARY`来指定列`j`为VARBINARY列。

```SQL
CREATE TABLE `test_binary` (
    `id` INT(11) NOT NULL COMMENT "",
    `j`  VARBINARY NULL COMMENT ""
) ENGINE=OLAP
DUPLICATE KEY(`id`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`id`)
PROPERTIES (
    "replication_num" = "3",
    "storage_format" = "DEFAULT"
);

mysql> DESC test_binary;
+-------+-----------+------+-------+---------+-------+
| Field | Type      | Null | Key   | Default | Extra |
+-------+-----------+------+-------+---------+-------+
| id    | int       | NO   | true  | NULL    |       |
| j     | varbinary | YES  | false | NULL    |       |
+-------+-----------+------+-------+---------+-------+
2 rows in set (0.01 sec)

```

### 加载数据并存储为BINARY类型

StarRocks支持以下方法将数据加载并存储为BINARY类型。

- 方法1：使用INSERT INTO将数据写入BINARY类型的常量列（如列`j`），常量列前缀为`x''`。

    ```SQL
    INSERT INTO test_binary (id, j) VALUES (1, x'abab');
    INSERT INTO test_binary (id, j) VALUES (2, x'baba');
    INSERT INTO test_binary (id, j) VALUES (3, x'010102');
    INSERT INTO test_binary (id, j) VALUES (4, x'0000'); 
    ```

- 方法2：使用[to_binary](../../sql-functions/binary-functions/to_binary.md)函数将VARCHAR数据转换为二进制数据。

    ```SQL
    INSERT INTO test_binary select 5, to_binary('abab', 'hex');
    INSERT INTO test_binary select 6, to_binary('abab', 'base64');
    INSERT INTO test_binary select 7, to_binary('abab', 'utf8');
    ```

- 方法3：使用Broker Load加载Parquet或ORC文件，并将文件存储为BINARY数据。有关更多信息，请参见[Broker Load](../data-manipulation/BROKER_LOAD.md)。

  - 对于Parquet文件，直接将`parquet::Type::type::BYTE_ARRAY`转换为`TYPE_VARBINARY`。
  - 对于ORC文件，直接将`orc::BINARY`转换为`TYPE_VARBINARY`。

- 方法4：使用Stream Load加载CSV文件，并将文件存储为`BINARY`类型数据。有关更多信息，请参见[Load CSV data](../../../loading/StreamLoad.md#load-csv-data)。
  - CSV文件使用十六进制格式表示二进制数据，请确保输入的二进制值是有效的十六进制值。
  - 仅支持CSV文件中的`BINARY`类型。JSON文件不支持`BINARY`类型。

  例如，`t1`是具有VARBINARY列`b`的表。

    ```sql
    CREATE TABLE `t1` (
    `k` int(11) NOT NULL COMMENT "",
    `v` int(11) NOT NULL COMMENT "",
    `b` varbinary
    ) ENGINE = OLAP
    DUPLICATE KEY(`k`)
    PARTITION BY RANGE(`v`) (
    PARTITION p1 VALUES [("-2147483648"), ("0")),
    PARTITION p2 VALUES [("0"), ("10")),
    PARTITION p3 VALUES [("10"), ("20")))
    DISTRIBUTED BY HASH(`k`)
    PROPERTIES ("replication_num" = "1");

    -- csv文件
    -- cat temp_data
    0,0,ab

    -- 使用Stream Load加载CSV文件。
    curl --location-trusted -u <username>:<password> -T temp_data -XPUT -H column_separator:, -H label:xx http://172.17.0.1:8131/api/test_mv/t1/_stream_load

    -- 查询加载的数据。
    mysql> select * from t1;
    +------+------+------------+
    | k    | v    | xx         |
    +------+------+------------+
    |    0 |    0 | 0xAB       |
    +------+------+------------+
    1 rows in set (0.11 sec)
    ```

### 查询和处理BINARY数据

StarRocks支持查询和处理BINARY数据，并支持使用BINARY函数和运算符。此示例使用表`test_binary`。

注意：当你从MySQL客户端访问StarRocks时，如果添加了`--binary-as-hex`选项，则二进制数据将以十六进制表示。

```Plain Text
mysql> select * from test_binary;
+------+------------+
| id   | j          |
+------+------------+
|    1 | 0xABAB     |
|    2 | 0xBABA     |
|    3 | 0x010102   |
|    4 | 0x0000     |
|    5 | 0xABAB     |
|    6 | 0xABAB     |
|    7 | 0x61626162 |
+------+------------+
7 rows in set (0.08 sec)
```

示例1：使用[hex](../../sql-functions/string-functions/hex.md)函数查看二进制数据。

```plain
mysql> select id, hex(j) from test_binary;
+------+----------+
| id   | hex(j)   |
+------+----------+
|    1 | ABAB     |
|    2 | BABA     |
|    3 | 010102   |
|    4 | 0000     |
|    5 | ABAB     |
|    6 | ABAB     |
|    7 | 61626162 |
+------+----------+
7 rows in set (0.02 sec)
```

示例2：使用[to_base64](../../sql-functions/crytographic-functions/to_base64.md)函数查看二进制数据。

```plain
mysql> select id, to_base64(j) from test_binary;
+------+--------------+
| id   | to_base64(j) |
+------+--------------+
|    1 | q6s=         |
|    2 | uro=         |
|    3 | AQEC         |
|    4 | AAA=         |
|    5 | q6s=         |
|    6 | q6s=         |
|    7 | YWJhYg==     |
+------+--------------+
7 rows in set (0.01 sec)
```

示例3：使用[from_binary](../../sql-functions/binary-functions/from_binary.md)函数查看二进制数据。

```plain
mysql> select id, from_binary(j, 'hex') from test_binary;
+------+-----------------------+
| id   | from_binary(j, 'hex') |
+------+-----------------------+
|    1 | ABAB                  |
|    2 | BABA                  |
|    3 | 010102                |
|    4 | 0000                  |
|    5 | ABAB                  |
|    6 | ABAB                  |
|    7 | 61626162              |
+------+-----------------------+
7 rows in set (0.01 sec)
```