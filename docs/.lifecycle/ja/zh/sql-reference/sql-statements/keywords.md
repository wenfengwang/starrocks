---
displayed_sidebar: Chinese
---

# キーワード

本文書では、非予約キーワードと予約キーワードについて説明し、StarRocksにおけるすべての予約キーワードを列挙しています。

## 简介

キーワードはSQL文において特別な意味を持ちます。例えば `CREATE` や `DROP` などです。以下の通りです：

- **非予約キーワード** (Non-reserved keywords) は、特別な処理を必要とせず、テーブル名やカラム名などの識別子として直接使用できます。例えば、DBは非予約キーワードで、`DB` という名前のデータベースを作成する場合、以下のような構文になります。

    ```SQL
    CREATE DATABASE DB;
    Query OK, 0 rows affected (0.00 sec)
    ```

- **予約キーワード** (Reserved keywords) は、変数や関数などの識別子として直接使用することはできず、特別な処理が必要です。StarRocksでは、予約キーワードを識別子として使用する場合、バッククォート (`\`) で囲む必要があります。例えば、`LIKE` は予約キーワードで、`LIKE` という名前のデータベースを作成する場合、以下のような構文になります。

    ```SQL
    CREATE DATABASE `LIKE`;
    Query OK, 0 rows affected (0.01 sec)
    ```

  バッククォート (`\`) で囲まない場合、エラーが発生します：

   ```SQL
    CREATE DATABASE LIKE;
    ERROR 1064 (HY000): Getting syntax error at line 1, column 16. Detail message: Unexpected input 'like', the most similar input is {a legal identifier}.
    ```

## 予約キーワード

以下は、StarRocksにおけるすべての予約キーワードをアルファベット順に列挙したものです。使用する際はバッククォート (`\`) で囲む必要があります。StarRocksのバージョンによって予約キーワードは異なる場合があります。

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
- COMPACTION (3.1 以降)
- CONVERT
- CREATE
- CROSS
- CUBE
- CURRENT_DATE
- CURRENT_ROLE (3.0 以降)
- CURRENT_TIME
- CURRENT_TIMESTAMP
- CURRENT_USER

### D

- DATABASE
- DATABASES
- DECIMAL
- DECIMALV2
- DECIMAL32
- DECIMAL64
- DECIMAL128
- DEFAULT
- DEFERRED (3.0 以降)
- DELETE
- DENSE_RANK
- DESC
- DESCRIBE
- DISTINCT
- DOUBLE
- DROP
- DUAL

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
- IMMEDIATE (3.0 以降)
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
- TEXT (3.1 以降)
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
