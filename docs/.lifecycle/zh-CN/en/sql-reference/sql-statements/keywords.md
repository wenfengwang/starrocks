---
displayed_sidebar: "Chinese"
---

# 关键词

本主题描述了StarRocks的非保留关键字和保留关键字。它提供了StarRocks中保留关键字的列表。

## 介绍

SQL语句中的关键字（例如`CREATE`和`DROP`）在StarRocks解析时具有特殊含义。关键字被分为非保留关键字和保留关键字。

- **非保留关键字** 可以直接用作标识符，无需特殊处理，例如表名和列名。例如，`DB`是非保留关键字。你可以创建一个名为`DB`的数据库。

    ```SQL
    CREATE DATABASE DB;
    查询成功，影响行数：0，耗时0.00秒
    ```

- **保留关键字** 只有经过特殊处理后才能用作标识符。例如，`LIKE`是一个保留关键字。如果要将其用作标识数据库，需要用一对反引号（`\`）将其括起来。

    ```SQL
    CREATE DATABASE `LIKE`;
    查询成功，影响行数：0，耗时0.01秒
    ```

  如果不用反引号（`\`）括起来，将返回错误：

    ```SQL
    CREATE DATABASE LIKE;
    错误1064（HY000）：在第1行，第16列获取了语法错误。详细消息：意外输入'like'，最相近的输入是{合法标识符}。
    ```

## 保留关键字

以下是StarRocks按字母顺序排列的保留关键字。如果要将其用作标识符，必须用反引号（`\`）括起来。保留关键字可能会随StarRocks版本变化。

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
- COMPACTION（v3.1及更高版本）
- CONVERT
- CREATE
- CROSS
- CUBE
- CURRENT_DATE
- CURRENT_TIME
- CURRENT_TIMESTAMP
- CURRENT_USER
- CURRENT_ROLE（v3.0及更高版本）

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
- DEFERRED（v3.0及更高版本）

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
- IMMEDIATE（v3.0及更高版本）

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
- TEXT（v3.1及更高版本）
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