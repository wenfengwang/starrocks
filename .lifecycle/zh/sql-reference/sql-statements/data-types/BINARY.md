---
displayed_sidebar: English
---

# 二进制/变长二进制

## 描述

BINARY(M)

VARBINARY(M)

自v3.0起，StarRocks支持BINARY/VARBINARY数据类型，用于存储二进制数据。支持的最大长度与VARCHAR（1~1048576）一致，单位为字节。若未指定M，则默认为1048576。二进制数据类型包含字节串，而字符数据类型包含字符串。

BINARY是VARBINARY的同义词，使用方法与VARBINARY相同。

## 限制与使用说明

- 在重复键、主键和唯一键表中支持VARBINARY列，但在聚合表中不支持。

- VARBINARY列不能用作分区键、桶键或在重复键、主键和唯一键表中的维度列。它们也不能用于ORDER BY、GROUP BY和JOIN子句。

- 当长度不对齐时，BINARY(M)/VARBINARY(M)不会进行右侧填充。

## 示例

### 创建VARBINARY类型列

创建表时，使用VARBINARY关键字为列j指定VARBINARY类型。

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

### 加载数据并以BINARY类型存储

StarRocks支持以下方法来加载数据并以BINARY类型存储：

- 方法一：使用INSERT INTO语句将数据写入BINARY类型的常量列（例如列j），其中常量列以x''作为前缀。

  ```SQL
  INSERT INTO test_binary (id, j) VALUES (1, x'abab');
  INSERT INTO test_binary (id, j) VALUES (2, x'baba');
  INSERT INTO test_binary (id, j) VALUES (3, x'010102');
  INSERT INTO test_binary (id, j) VALUES (4, x'0000'); 
  ```

- 方法二：使用[to_binary](../../sql-functions/binary-functions/to_binary.md)函数将VARCHAR数据转换为二进制数据。

  ```SQL
  INSERT INTO test_binary select 5, to_binary('abab', 'hex');
  INSERT INTO test_binary select 6, to_binary('abab', 'base64');
  INSERT INTO test_binary select 7, to_binary('abab', 'utf8');
  ```

- 方法三：使用Broker Load加载Parquet或ORC文件，并将文件存储为BINARY数据。更多信息请参见[Broker Load](../data-manipulation/BROKER_LOAD.md)文档。

  - 对于Parquet文件，直接将parquet::Type::type::BYTE_ARRAY转换为TYPE_VARBINARY。
  - 对于ORC文件，直接将orc::BINARY转换为TYPE_VARBINARY。

- 方法 4: 使用Stream Load加载CSV文件并将文件存储为`BINARY`数据。更多信息请参见[Load CSV data](../../../loading/StreamLoad.md#load-csv-data)。
  - CSV文件将二进制数据以十六进制格式表示，请确保输入的二进制值为有效的十六进制值。
  - BINARY类型仅在CSV文件中受支持，JSON文件不支持BINARY类型。

  例如，t1是一张包含VARBINARY列b的表。

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
  
  -- csv file
  -- cat temp_data
  0,0,ab
  
  -- Load CSV file using Stream Load.
  curl --location-trusted -u <username>:<password> -T temp_data -XPUT -H column_separator:, -H label:xx http://172.17.0.1:8131/api/test_mv/t1/_stream_load
  
  -- Query the loaded data.
  mysql> select * from t1;
  +------+------+------------+
  | k    | v    | xx         |
  +------+------+------------+
  |    0 |    0 | 0xAB       |
  +------+------+------------+
  1 rows in set (0.11 sec)
  ```

### 查询与处理BINARY数据

StarRocks支持查询和处理BINARY数据，并支持使用BINARY函数和操作符。以下示例使用名为test_binary的表。

注意：在使用MySQL客户端访问StarRocks时，如果添加了--binary-as-hex选项，二进制数据将以十六进制形式显示。

```Plain
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

示例 1: 使用[hex](../../sql-functions/string-functions/hex.md)函数查看二进制数据。

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

示例 2: View binary data using the [to_base64](../../sql-functions/crytographic-functions/to_base64.md) function.

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

示例 3: View binary data using the [from_binary](../../sql-functions/binary-functions/from_binary.md) function.

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
