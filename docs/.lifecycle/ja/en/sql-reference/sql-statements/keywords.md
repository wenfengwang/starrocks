---
displayed_sidebar: English
---

# キーワード

このトピックでは、予約されていないキーワードと予約済みキーワードについて説明します。StarRocksの予約済みキーワードのリストを提供します。

## 紹介

SQL ステートメント内のキーワード（`CREATE` や `DROP` など）は、StarRocks によって解析されたときに特別な意味を持ちます。キーワードは、予約されていないキーワードと予約済みキーワードに分類されます。

- **予約されていないキーワード** は、特別な処理を行わずに、識別子として直接使用できます。例えば、`DB` は予約されていないキーワードです。`DB` という名前のデータベースを作成できます。

    ```SQL
    CREATE DATABASE DB;
    Query OK, 0 rows affected (0.00 sec)
    ```

- **予約済みキーワード** は、特別な処理を施した後にのみ識別子として使用できます。例えば、`LIKE` は予約済みキーワードです。データベースの識別子として使用する場合は、バッククォート（`\`）で囲みます。

    ```SQL
    CREATE DATABASE `LIKE`;
    Query OK, 0 rows affected (0.01 sec)
    ```

  バッククォート（`\`）で囲まれていない場合、エラーが返されます。

    ```SQL
    CREATE DATABASE LIKE;
    ERROR 1064 (HY000): Getting syntax error at line 1, column 16. Detail message: Unexpected input 'like', the most similar input is {a legal identifier}.
    ```

## 予約済みキーワード

以下は、StarRocks の予約キーワードをアルファベット順に並べたものです。識別子として使用する場合は、バッククォート（`\`）で囲む必要があります。予約キーワードは、StarRocks のバージョンによって異なる場合があります。

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
- COMPACTION (v3.1 以降)
- CONVERT
- CREATE
- CROSS
- CUBE
- CURRENT_DATE
- CURRENT_TIME
- CURRENT_TIMESTAMP
- CURRENT_USER
- CURRENT_ROLE (v3.0 以降)

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
- DEFERRED (v3.0 以降)

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
- IMMEDIATE (v3.0 以降)

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
- TEXT (v3.1 以降)
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
