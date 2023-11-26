---
displayed_sidebar: "Japanese"
---

# キーワード

このトピックでは、非予約キーワードと予約キーワードについて説明します。StarRocksで予約されたキーワードのリストを提供します。

## 紹介

SQLステートメントの中で、`CREATE`や`DROP`などのキーワードは、StarRocksによって解析される際に特別な意味を持ちます。キーワードは非予約キーワードと予約キーワードに分類されます。

- **非予約キーワード**は、特別な処理なしに識別子として直接使用することができます。例えば、`DB`は非予約キーワードです。`DB`という名前のデータベースを作成することができます。

    ```SQL
    CREATE DATABASE DB;
    クエリ OK、0 行が変更されました (0.00 秒)
    ```

- **予約キーワード**は、特別な処理を行った後に識別子として使用することができます。例えば、`LIKE`は予約キーワードです。データベースを識別するために使用する場合は、バッククォート（`\`）で囲んでください。

    ```SQL
    CREATE DATABASE `LIKE`;
    クエリ OK、0 行が変更されました (0.01 秒)
    ```

  バッククォート（`\`）で囲まれていない場合、エラーが返されます。

    ```SQL
    CREATE DATABASE LIKE;
    ERROR 1064 (HY000): Getting syntax error at line 1, column 16. Detail message: Unexpected input 'like', the most similar input is {a legal identifier}.
    ```

## 予約キーワード

以下は、StarRocksの予約キーワードのアルファベット順に並べられたリストです。これらを識別子として使用する場合は、バッククォート（`\`）で囲む必要があります。予約キーワードはStarRocksのバージョンによって異なる場合があります。

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
- COMPACTION（v3.1以降）
- CONVERT
- CREATE
- CROSS
- CUBE
- CURRENT_DATE
- CURRENT_TIME
- CURRENT_TIMESTAMP
- CURRENT_USER
- CURRENT_ROLE（v3.0以降）

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
- DEFERRED（v3.0以降）

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
- IMMEDIATE（v3.0以降）

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
- TEXT（v3.1以降）
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
