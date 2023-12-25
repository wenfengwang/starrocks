---
displayed_sidebar: Chinese
---

# cardinality

## 機能

Map 中の要素の数を計算し、戻り値の型は INT です。MAP にはキーと値のペア (key-value pair) が保存されています。例えば `{"a":1, "b":2}` のように。一つのキーと値のペアを一つの要素とし、`{"a":1, "b":2}` の要素数は 2 です。

この関数はバージョン 3.0 からサポートされています。関数の別名は [map_size](map_size.md) です。

## 文法

```Haskell
INT cardinality(any_map)
```

## パラメータ説明

`any_map`: 要素の数を取得する MAP の値。

## 戻り値の説明

INT 型の値を返します。入力パラメータが NULL の場合、結果も NULL です。

MAP の Key と Value は NULL でも構いません。正常に計算されます。

## 例

### StarRocks のローカルテーブルの MAP データを照会する

バージョン 3.1 では、テーブル作成時に MAP 型の列を定義することがサポートされています。`test_map` テーブルの作成を例にします。

```SQL
CREATE TABLE test_map(
    col_int INT,
    col_map MAP<VARCHAR(50),INT>
  )
DUPLICATE KEY(col_int);

INSERT INTO test_map VALUES
(1,map{"a":1,"b":2}),
(2,map{"c":3}),
(3,map{"d":4,"e":5});

SELECT * FROM test_map ORDER BY col_int;
+---------+---------------+
| col_int | col_map       |
+---------+---------------+
|       1 | {"a":1,"b":2} |
|       2 | {"c":3}       |
|       3 | {"d":4,"e":5} |
+---------+---------------+
```

`col_map` 列の各行の要素数を計算します。

```Plain
SELECT cardinality(col_map) FROM test_map ORDER BY col_int;
+----------------------+
| cardinality(col_map) |
+----------------------+
|                 2    |
|                 1    |
|                 2    |
+----------------------+
```

### 外部データレイクの MAP データを照会する

Hive に `hive_map` というテーブルがあると仮定し、以下のデータがあります：

```Plain
SELECT * FROM hive_map ORDER BY col_int;
+---------+---------------+
| col_int | col_map       |
+---------+---------------+
|       1 | {"a":1,"b":2} |
|       2 | {"c":3}       |
|       3 | {"d":4,"e":5} |
+---------+---------------+
3 rows in set (0.05 sec)
```

StarRocks クラスタで [Hive catalog を作成する](../../../data_source/catalog/hive_catalog.md#创建-hive-catalog) ことにより、このテーブルにアクセスし、`col_map` 列の各行の要素数を計算します。

```Plaintext
SELECT cardinality(col_map) FROM hive_map ORDER BY col_int;
+----------------------+
| cardinality(col_map) |
+----------------------+
|                    2 |
|                    1 |
|                    2 |
+----------------------+
3 rows in set (0.05 sec)
```
