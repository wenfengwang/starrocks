---
displayed_sidebar: English
---

# BINARY/VARBINARY

## 説明

BINARY(M)

VARBINARY(M)

v3.0以降、StarRocksはバイナリデータの格納に使用されるBINARY/VARBINARYデータ型をサポートしています。サポートされる最大長は、VARCHAR (1〜1048576) と同じです。単位はバイトです。`M`が指定されていない場合、デフォルトで1048576が使用されます。バイナリデータ型にはバイト文字列が含まれ、文字データ型には文字列が含まれます。

BINARYはVARBINARYのエイリアスです。使用方法はVARBINARYと同じです。

## 制限と使用上の注意

- VARBINARYカラムは、Duplicate Key、Primary Key、Unique Keyテーブルでサポートされています。Aggregateテーブルではサポートされていません。

- VARBINARYカラムは、パーティションキー、バケットキー、またはDuplicate Key、Primary Key、Unique Keyテーブルのディメンション列として使用できません。ORDER BY、GROUP BY、JOIN句では使用できません。

- BINARY(M)/VARBINARY(M)は、長さが一致しない場合に右詰めされません。

## 例

### VARBINARY型の列を作成する

テーブルを作成する際に、キーワード`VARBINARY`を使用してカラム`j`をVARBINARY列として指定します。

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

### データをロードしてBINARY型として保存する

StarRocksは、以下の方法でデータをロードしてBINARY型として保存することをサポートしています。

- 方法 1: INSERT INTOを使用して、BINARY型の定数カラム（例えばカラム`j`）にデータを書き込む場合、定数カラムは`x''`でプレフィックスされます。

    ```SQL
    INSERT INTO test_binary (id, j) VALUES (1, x'abab');
    INSERT INTO test_binary (id, j) VALUES (2, x'baba');
    INSERT INTO test_binary (id, j) VALUES (3, x'010102');
    INSERT INTO test_binary (id, j) VALUES (4, x'0000'); 
    ```

- 方法 2: [to_binary](../../sql-functions/binary-functions/to_binary.md)関数を使用してVARCHARデータをバイナリデータに変換します。

    ```SQL
    INSERT INTO test_binary SELECT 5, to_binary('abab', 'hex');
    INSERT INTO test_binary SELECT 6, to_binary('abab', 'base64');
    INSERT INTO test_binary SELECT 7, to_binary('abab', 'utf8');
    ```

- 方法 3: Broker Loadを使用してParquetまたはORCファイルをロードし、ファイルをBINARYデータとして保存します。詳細は[Broker Load](../data-manipulation/BROKER_LOAD.md)を参照してください。

  - Parquetファイルでは、`parquet::Type::type::BYTE_ARRAY`を直接`TYPE_VARBINARY`に変換します。
  - ORCファイルでは、`orc::BINARY`を直接`TYPE_VARBINARY`に変換します。

- 方法 4: Stream Loadを使用してCSVファイルをロードし、ファイルを`BINARY`データとして保存します。詳細は[CSVデータのロード](../../../loading/StreamLoad.md#load-csv-data)を参照してください。
  - CSVファイルはバイナリデータに16進数形式を使用します。入力バイナリ値が有効な16進数であることを確認してください。
  - `BINARY`型はCSVファイルでのみサポートされています。JSONファイルでは`BINARY`型はサポートされていません。

  例えば、`t1`はVARBINARYカラム`b`を持つテーブルです。

    ```sql
    CREATE TABLE `t1` (
    `k` int(11) NOT NULL COMMENT "",
    `v` int(11) NOT NULL COMMENT "",
    `b` varbinary
    ) ENGINE = OLAP
    DUPLICATE KEY(`k`)
    PARTITION BY RANGE(`v`) (
    PARTITION p1 VALUES LESS THAN ("0"),
    PARTITION p2 VALUES LESS THAN ("10"),
    PARTITION p3 VALUES LESS THAN ("20"))
    DISTRIBUTED BY HASH(`k`)
    PROPERTIES ("replication_num" = "1");

    -- csv file
    -- cat temp_data
    0,0,ab

    -- CSVファイルをStream Loadを使用してロードします。
    curl --location-trusted -u <username>:<password> -T temp_data -XPUT -H "column_separator:," -H "label:xx" http://172.17.0.1:8131/api/test_mv/t1/_stream_load

    -- ロードされたデータをクエリします。
    mysql> SELECT * FROM t1;
    +------+------+------------+
    | k    | v    | b          |
    +------+------+------------+
    |    0 |    0 | 0xAB       |
    +------+------+------------+
    1 rows in set (0.11 sec)
    ```

### BINARYデータのクエリと処理

StarRocksはBINARYデータのクエリと処理をサポートし、BINARY関数と演算子の使用をサポートしています。この例では、テーブル`test_binary`を使用します。

注: MySQLクライアントからStarRocksにアクセスする際に`--binary-as-hex`オプションを追加すると、バイナリデータは16進数表記で表示されます。

```Plain Text
mysql> SELECT * FROM test_binary;
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

例 1: [hex](../../sql-functions/string-functions/hex.md)関数を使用してバイナリデータを表示します。

```plain
mysql> SELECT id, hex(j) FROM test_binary;
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

例 2: [to_base64](../../sql-functions/cryptographic-functions/to_base64.md)関数を使用してバイナリデータを表示します。

```plain
mysql> SELECT id, to_base64(j) FROM test_binary;
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

例 3: [from_binary](../../sql-functions/binary-functions/from_binary.md)関数を使用してバイナリデータを表示します。

```plain
mysql> SELECT id, from_binary(j, 'hex') FROM test_binary;
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
