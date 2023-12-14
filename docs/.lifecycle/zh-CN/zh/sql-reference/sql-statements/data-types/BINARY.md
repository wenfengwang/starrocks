---
displayed_sidebar: "Chinese"
---

# BINARY/VARBINARY

## Description

BINARY(M)

VARBINARY(M)

Starting from version 3.0, StarRocks supports the BINARY/VARBINARY data types for storing binary data, measured in bytes.

The maximum supported length is the same as the VARCHAR type, with the value of `M` ranging from 1 to 1048576. If `M` is not specified, the default value is the maximum value of 1048576.

BINARY is an alias for VARBINARY and can be used interchangeably with VARBINARY.

## Restrictions and Notices

- VARBINARY types of columns can be created in the detail, primary key, and update model tables, but not in the aggregate model table.
- Columns of BINARY/VARBINARY types are not supported as partition keys, bucketing keys, or dimension columns in detail, primary key, and update model tables. They also cannot be used in the JOIN, GROUP BY, or ORDER BY clauses.
- BINARY(M)/VARBINARY(M) does not pad the length if it is not aligned.

## Examples

### Creating a BINARY type column

When creating a table, use the keyword `VARBINARY` to specify the column `j` as a VARBINARY type.

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

### Importing and storing data as BINARY type

StarRocks supports the following methods to import and store data as BINARY type.

- Method 1: Use INSERT INTO to write data into the constant column of BINARY type (e.g. column `j`), with the constant column prefixed by `x''`.

```SQL
INSERT INTO test_binary (id, j) VALUES (1, x'abab');
INSERT INTO test_binary (id, j) VALUES (2, x'baba');
INSERT INTO test_binary (id, j) VALUES (3, x'010102');
INSERT INTO test_binary (id, j) VALUES (4, x'0000');
```

- Method 2: Use the TO_BINARY() function to convert VARCHAR type data to BINARY type.

```SQL
INSERT INTO test_binary select 5, to_binary('abab', 'hex');
INSERT INTO test_binary select 6, to_binary('abab', 'base64');
INSERT INTO test_binary select 7, to_binary('abab', 'utf8');
```

### Querying and processing BINARY type data

StarRocks supports querying and processing BINARY type data, as well as using BINARY functions and operators. This example uses the `test_binary` table to illustrate.

> Note: If the `--binary-as-hex` option is added when starting the MySQL client, the BINARY data in the result will be displayed in `hex` format by default.

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

Example 1: Use the [hex](../../sql-functions/string-functions/hex.md) function to view BINARY type data.

```Plain Text
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