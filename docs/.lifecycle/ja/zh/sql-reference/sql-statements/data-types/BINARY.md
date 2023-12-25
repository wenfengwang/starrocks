---
displayed_sidebar: Chinese
---

# BINARY/VARBINARY

## 説明

BINARY(M)

VARBINARY(M)

バージョン 3.0 から、StarRocks は BINARY/VARBINARY データ型をサポートしており、二進数データの格納に使用されます。単位はバイトです。

サポートされる最大長は VARCHAR 型と同じで、`M` の値の範囲は 1~1048576 です。`M` が指定されていない場合、デフォルトは最大値の 1048576 になります。

BINARY は VARBINARY の別名で、使用方法は VARBINARY と同じです。

## 制限と注意事項

- 明細モデル、主キーモデル、更新モデルのテーブルに VARBINARY 型の列を作成することをサポートしていますが、集約モデルのテーブルには VARBINARY 型の列を作成することはサポートしていません。
- BINARY/VARBINARY 型の列を明細モデル、主キーモデル、更新モデルのテーブルのパーティションキー、バケットキー、次元列として使用すること、また JOIN、GROUP BY、ORDER BY 句での使用は現在サポートされていません。
- BINARY(M)/VARBINARY(M) は、未揃えの長さに対してパディングを行いません。

## 例

### BINARY 型の列を作成する

テーブルを作成する際に、キーワード `VARBINARY` を使用して、列 `j` を VARBINARY 型として指定します。

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

### データをインポートして BINARY 型として保存する

StarRocks は以下の方法でデータをインポートし、BINARY 型として保存することをサポートしています。

- 方法1：INSERT INTO を使用して、BINARY 型の定数列（例えば列 `j`）にデータを書き込みます。定数列は `x''` でプレフィックスされます。

```SQL
INSERT INTO test_binary (id, j) VALUES (1, x'abab');
INSERT INTO test_binary (id, j) VALUES (2, x'baba');
INSERT INTO test_binary (id, j) VALUES (3, x'010102');
INSERT INTO test_binary (id, j) VALUES (4, x'0000');
```

- 方法2：TO_BINARY() 関数を使用して、VARCHAR 型のデータを BINARY 型に変換します。

```SQL
INSERT INTO test_binary select 5, to_binary('abab', 'hex');
INSERT INTO test_binary select 6, to_binary('abab', 'base64');
INSERT INTO test_binary select 7, to_binary('abab', 'utf8');
```

### BINARY 型のデータをクエリして処理する

StarRocks は BINARY 型のデータのクエリと処理をサポートし、BINARY 関数と演算子の使用をサポートしています。この例では `test_binary` テーブルを使用して説明します。

> 注意：MySQL クライアントを `--binary-as-hex` オプション付きで起動した場合、BINARY データはデフォルトで `hex` 形式で表示されます。

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

例1：[hex](../../sql-functions/string-functions/hex.md) 関数を使用して BINARY 型のデータを表示します。

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
