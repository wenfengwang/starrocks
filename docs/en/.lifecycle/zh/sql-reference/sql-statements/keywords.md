---
displayed_sidebar: English
---

# 关键词

本主题描述了非保留关键词和保留关键词。它提供了 StarRocks 中的保留关键词列表。

## 介绍

SQL 语句中的关键词，例如 `CREATE` 和 `DROP`，在 StarRocks 解析时具有特殊含义。关键词分为非保留关键词和保留关键词。

- **非保留关键词** 可以直接用作标识符，无需特殊处理，例如表名和列名。例如，`DB` 是一个非保留关键词。您可以创建一个名为 `DB` 的数据库。

    ```SQL
    CREATE DATABASE DB;
    Query OK, 0 rows affected (0.00 sec)
    ```

- **保留关键词** 经过特殊处理后才能用作标识符。例如，`LIKE` 是一个保留关键词。如果要将其用于标识数据库，请将其括在一对反引号（`\`）中。

    ```SQL
    CREATE DATABASE `LIKE`;
    Query OK, 0 rows affected (0.01 sec)
    ```

  如果没有将其括在反引号（`\`）中，则会返回错误：

    ```SQL
    CREATE DATABASE LIKE;
    ERROR 1064 (HY000): 在第 1 行、第 16 列附近有语法错误。详细信息：意外输入 'like'，最相似的输入是 {合法标识符}。
    ```

## 保留关键词

以下是 StarRocks 保留关键词按字母顺序排列的列表。如果要将它们用作标识符，则必须将它们括在反引号（`\`）中。保留关键词可能会随 StarRocks 版本的不同而有所变化。

### A

- ADD
- ALL
- ALTER
- ANALYZE
- AND
- ARRAY
- AS
- ASC

### B

- BETWEEN
- BIGINT
- BITMAP
- BOTH
- BY

### C

- CASE
- CHAR
- CHARACTER
- CHECK
- COLLATE
- COLUMN
- COMPACTION（v3.1 及更高版本）
- CONVERT
- CREATE
- CROSS
- CUBE
- CURRENT_DATE
- CURRENT_TIME
- CURRENT_TIMESTAMP
- CURRENT_USER
- CURRENT_ROLE（v3.0 及更高版本）

### D

- DATABASE
- DATABASES
- DECIMAL
- DECIMALV2
- DECIMAL32
- DECIMAL64
- DECIMAL128
- DEFAULT
- DELETE
- DENSE_RANK
- DESC
- DESCRIBE
- DISTINCT
- DOUBLE
- DROP
- DUAL
- DEFERRED（v3.0 及更高版本）

### E

- ELSE
- EXCEPT
- EXISTS
- EXPLAIN

### F

- FALSE
- FIRST_VALUE
- FLOAT
- FOR
- FORCE
- FROM
- FULL
- FUNCTION

### G

- GRANT
- GROUP
- GROUPS
- GROUPING
- GROUPING_ID

### H

- HAVING
- HLL
- HOST

### I

- IF
- IGNORE
- IN
- INDEX
- INFILE
- INNER
- INSERT
- INT
- INTEGER
- INTERSECT
- INTO
- IS
- IMMEDIATE（v3.0 及更高版本）

### J

- JOIN
- JSON

### K

- KEY
- KEYS
- KILL

### L

- LAG
- LARGEINT
- LAST_VALUE
- LATERAL
- LEAD
- LEFT
- LIKE
- LIMIT
- LOAD
- LOCALTIME
- LOCALTIMESTAMP

### M

- MAXVALUE
- MINUS
- MOD

### N

- NTILE
- NOT
- NULL

### O

- ON
- OR
- ORDER
- OUTER
- OUTFILE
- OVER

### P

- PARTITION
- PERCENTILE
- PRIMARY
- PROCEDURE

### Q

- QUALIFY

### R

- RANGE
- RANK
- READ
- REGEXP
- RELEASE
- RENAME
- REPLACE
- REVOKE
- RIGHT
- RLIKE
- ROW
- ROWS
- ROW_NUMBER

### S

- SCHEMA
- SCHEMAS
- SELECT
- SET
- SET_VAR
- SHOW
- SMALLINT
- SYSTEM

### T

- TABLE
- TERMINATED
- TEXT（v3.1 及更高版本）
- THEN
- TINYINT
- TO
- TRUE

### U

- UNION
- UNIQUE
- UNSIGNED
- UPDATE
- USE
- USING

### V

- VALUES
- VARCHAR

### W

- WHEN
- WHERE
- WITH
