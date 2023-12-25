---
displayed_sidebar: Chinese
---

# map_size

## 機能

Map内の要素の数を計算します。Mapはキーと値のペア（key-value pair）を保存しており、例えば `{"a":1, "b":2}` のようになります。一つのキーと値のペアを一つの要素として数え、`{"a":1, "b":2}` の要素数は2です。

この関数の別名は [cardinality](cardinality.md) です。

この関数はバージョン2.5からサポートされています。

## 文法

```Haskell
map_size(any_map)
```

## パラメータ説明

`any_map`: 要素の数を取得したいMapの値。

## 戻り値の説明

INT型の値を返します。入力パラメータがNULLの場合、結果もNULLになります。

Map内のKeyとValueはNULLでも構いません。正常に計算されます。

## 例

### StarRocksのローカルテーブルでのMapデータのクエリ

バージョン3.1では、テーブル作成時にMap型の列を定義することがサポートされています。`test_map` テーブルを作成する例を示します。

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
SELECT map_size(col_map) FROM test_map ORDER BY col_int;
+-------------------+
| map_size(col_map) |
+-------------------+
|                 2 |
|                 1 |
|                 2 |
+-------------------+
```

### 外部データレイクでのMapデータのクエリ

Hiveに `hive_map` というテーブルがあると仮定し、以下のデータがあります:

```Plain
select * from hive_map order by col_int;
+---------+---------------+
| col_int | col_map       |
+---------+---------------+
|       1 | {"a":1,"b":2} |
|       2 | {"c":3}       |
|       3 | {"d":4,"e":5} |
+---------+---------------+
3 rows in set (0.05 sec)
```

StarRocksクラスターで[Hive catalogを作成](../../../data_source/catalog/hive_catalog.md#创建-hive-catalog)することにより、このテーブルにアクセスし、`col_map` 列の各行の要素数を計算します。

```Plaintext
select map_size(col_map) from hive_map order by col_int;
+-------------------+
| map_size(col_map) |
+-------------------+
|                 2 |
|                 1 |
|                 2 |
+-------------------+
3 rows in set (0.05 sec)
```
