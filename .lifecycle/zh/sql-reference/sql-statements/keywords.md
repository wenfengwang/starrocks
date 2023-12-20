---
displayed_sidebar: English
---

# 关键词

本主题介绍非保留关键词和保留关键词。它提供了 StarRocks 中的保留关键词列表。

## 简介

在 StarRocks 解析 SQL 语句时，如 CREATE 和 DROP 等关键词具有特殊的含义。关键词分为非保留关键词和保留关键词。

- **非保留关键词**可以直接用作标识符，不需要特殊处理，如表名和列名。例如，`DB` 是一个非保留关键词。您可以创建一个数据库名为 `DB`。

  ```SQL
  CREATE DATABASE DB;
  Query OK, 0 rows affected (0.00 sec)
  ```

- **保留关键词**只有在特殊处理后才能用作标识符。例如，`LIKE` 是一个保留关键词。如果您想用它来命名数据库，需要将其用反引号 (`\`) 括起来。

  ```SQL
  CREATE DATABASE `LIKE`;
  Query OK, 0 rows affected (0.01 sec)
  ```

  如果未用反引号 (`) 括起，则会返回错误：

  ```SQL
  CREATE DATABASE LIKE;
  ERROR 1064 (HY000): Getting syntax error at line 1, column 16. Detail message: Unexpected input 'like', the most similar input is {a legal identifier}.
  ```

## 保留关键词

以下是按字母顺序排列的 StarRocks 保留关键词。如果您想将它们用作标识符，必须用反引号 (`) 将它们括起来。保留关键词可能会随 StarRocks 版本的不同而有所变化。

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
- COMPACTION (v3.1 及以后版本)
- CONVERT
- CREATE
- CROSS
- CUBE
- CURRENT_DATE
- CURRENT_TIME
- CURRENT_TIMESTAMP
- CURRENT_USER
- CURRENT_ROLE (v3.0 及以后版本)

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
- DEFERRED (v3.0 及以后版本)

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
- IMMEDIATE (v3.0 及以后版本)

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
- PERCENT
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
- TEXT (v3.1 及以后版本)
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
