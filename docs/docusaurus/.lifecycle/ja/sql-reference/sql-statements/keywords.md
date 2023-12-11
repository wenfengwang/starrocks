---
displayed_sidebar: "Japanese"
---

# キーワード

このトピックでは、非予約キーワードと予約キーワードについて説明します。StarRocksの予約キーワードのリストを提供します。

## はじめに

`CREATE`や`DROP`などのSQL文中のキーワードは、StarRocksによって解析されるときに特別な意味を持ちます。これらのキーワードは非予約キーワードと予約キーワードに分類されます。

- **非予約キーワード** は、特別な処理をせずに、例えばテーブル名や列名として直接使用できます。例えば、`DB`は非予約キーワードです。`DB`という名前のデータベースを作成することができます。

    ```SQL
    CREATE DATABASE DB;
    クエリが実行されました：行に影響を受けませんでした（0.00秒）
    ```

- **予約キーワード** は特別な処理をした後に、識別子として使用することができます。例えば、`LIKE`は予約キーワードです。もしデータベースを識別するために使用したい場合は、バッククォート（`）で囲んでください。

    ```SQL
    CREATE DATABASE `LIKE`;
    クエリが実行されました：行に影響を受けませんでした（0.01秒）
    ```

  バッククォート（`）で囲まない場合、エラーが返されます。

    ```SQL
    CREATE DATABASE LIKE;
    ERROR 1064 (HY000): Getting syntax error at line 1, column 16. Detail message: Unexpected input 'like', the most similar input is {a legal identifier}.
    ```

## 予約キーワード

以下は、StarRocksの予約キーワードがアルファベット順に並べられています。これらを識別子として使用したい場合は、バッククォート（`）で囲む必要があります。予約キーワードはStarRocksのバージョンによって異なる場合があります。

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

（...以下略）