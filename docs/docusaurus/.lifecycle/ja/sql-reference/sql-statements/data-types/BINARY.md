```yaml
---
displayed_sidebar: "Japanese"
---

# BINARY/VARBINARY

## 説明

BINARY(M)

VARBINARY(M)

v3.0以降、StarRocksはバイナリデータを保存するために使用されるBINARY/VARBINARYデータ型をサポートしています。サポートされる最大長はVARCHARと同じです(1〜1048576)。単位はバイトです。`M`が指定されていない場合、デフォルトで1048576が使用されます。バイナリデータ型はバイト文字列を含み、文字列データ型は文字列を含みます。

BINARYはVARBINARYのエイリアスです。使用法はVARBINARYと同じです。

## 制限および使用上の注意

- VARBINARY列は、重複キー、プライマリキー、およびユニークキーのテーブルでサポートされています。これらは集約テーブルではサポートされていません。

- VARBINARY列は、重複キー、プライマリキー、およびユニークキーのテーブルのパーティションキー、バケット化キー、またはディメンション列として使用することはできません。ORDER BY、GROUP BY、JOIN句で使用することはできません。

- BINARY(M)/VARBINARY(M)は、アライメントされていない長さの場合には右側にパディングされません。

## 例

### VARBINARY型の列を作成する

テーブルを作成する際、キーワード`VARBINARY`を使用して列`j`をVARBINARY列として指定します。

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

### データの読み込みとBINARY型への格納

StarRocksはデータをロードしてBINARY型として保存するための以下の方法をサポートしています。

- 方法1：INSERT INTOを使用してBINARY型の定数列（例えば列`j`）にデータを書き込む際、定数列の前に`x''`を付けます。

    ```SQL
    INSERT INTO test_binary (id, j) VALUES (1, x'abab');
    INSERT INTO test_binary (id, j) VALUES (2, x'baba');
    INSERT INTO test_binary (id, j) VALUES (3, x'010102');
    INSERT INTO test_binary (id, j) VALUES (4, x'0000'); 
    ```

- 方法2：[to_binary](../../sql-functions/binary-functions/to_binary.md)関数を使用してVARCHARデータをバイナリデータに変換します。

    ```SQL
    INSERT INTO test_binary select 5, to_binary('abab', 'hex');
    INSERT INTO test_binary select 6, to_binary('abab', 'base64');
    INSERT INTO test_binary select 7, to_binary('abab', 'utf8');
    ```

- 方法3：ブローカーロードを使用してParquetファイルまたはORCファイルをロードし、ファイルをBINARYデータとして保存します。詳細は[BROKER_LOAD](../data-manipulation/BROKER_LOAD.md)を参照してください。

  - Parquetファイルでは、`parquet::Type::type::BYTE_ARRAY`を直接`TYPE_VARBINARY`に変換します。
  - ORCファイルでは、`orc::BINARY`を直接`TYPE_VARBINARY`に変換します。

- 方法4：ストリームロードを使用してCSVファイルをロードし、ファイルを`BINARY`データとして保存します。詳細は[CSVデータのロード](../../../loading/StreamLoad.md#load-csv-data)を参照してください。
  - CSVファイルではバイナリデータに対して16進数形式が使用されます。入力バイナリ値が有効な16進数値であることを確認してください。
  - `BINARY`型はCSVファイルのみでサポートされています。JSONファイルは`BINARY`型をサポートしていません。

  例えば、`t1`はVARBINARY列`b`を持つテーブルです。

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

    -- csvファイル
    -- cat temp_data
    0,0,ab

    -- ストリームロードを使用してCSVファイルをロードします。
    curl --location-trusted -u <username>:<password> -T temp_data -XPUT -H column_separator:, -H label:xx http://172.17.0.1:8131/api/test_mv/t1/_stream_load

    -- ロードされたデータをクエリします。
    mysql> select * from t1;
    +------+------+------------+
    | k    | v    | xx         |
    +------+------+------------+
    |    0 |    0 | 0xAB       |
    +------+------+------------+
    1 rows in set (0.11 sec)
    ```

### BINARYデータのクエリと処理

StarRocksはBINARYデータのクエリと処理をサポートし、BINARY関数と演算子の使用をサポートしています。この例では、テーブル`test_binary`を使用します。

注意: MySQLクライアントからStarRocksにアクセスする際に`--binary-as-hex`オプションを追加すると、バイナリデータは16進数表記を使用して表示されます。

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

例1：[hex](../../sql-functions/string-functions/hex.md)関数を使用してバイナリデータを表示します。

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

例2：[to_base64](../../sql-functions/crytographic-functions/to_base64.md)関数を使用してバイナリデータを表示します。

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

例3：[from_binary](../../sql-functions/binary-functions/from_binary.md)関数を使用してバイナリデータを表示します。

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