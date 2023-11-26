---
displayed_sidebar: "Japanese"
---

# map_size

## 説明

MAPの要素数を返します。MAPはキーと値のペアの順序のないコレクションであり、例えば `{"a":1, "b":2}` のような形式です。1つのキーと値のペアが1つの要素となり、例えば `{"a":1, "b":2}` は2つの要素を含んでいます。

この関数には [cardinality()](cardinality.md) という別名があります。v2.5からサポートされています。

## 構文

```Haskell
INT map_size(any_map)
```

## パラメータ

`any_map`: 要素数を取得したいMAPの値です。

## 戻り値

INT型の値を返します。

入力がNULLの場合、NULLが返されます。

MAPのキーまたは値がNULLの場合、NULLは通常の値として処理されます。

## 例

### StarRocksネイティブテーブルからMAPデータをクエリする

v3.1以降、StarRocksではテーブルを作成する際にMAP列を定義することがサポートされています。この例では、以下のデータを含む `test_map` テーブルを使用します。

```Plain
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
3 rows in set (0.05 sec)
```

`col_map` 列の各行の要素数を取得します。

```Plaintext
select map_size(col_map) from test_map order by col_int;
+-------------------+
| map_size(col_map) |
+-------------------+
|                 2 |
|                 1 |
|                 2 |
+-------------------+
3 rows in set (0.05 sec)
```

### データレイクからMAPデータをクエリする

この例では、以下のデータを含むHiveテーブル `hive_map` を使用します。

```Plaintext
SELECT * FROM hive_map ORDER BY col_int;
+---------+---------------+
| col_int | col_map       |
+---------+---------------+
|       1 | {"a":1,"b":2} |
|       2 | {"c":3}       |
|       3 | {"d":4,"e":5} |
+---------+---------------+
```

[Hiveカタログ](../../../data_source/catalog/hive_catalog.md#create-a-hive-catalog)がクラスタに作成された後、このカタログとmap_size()関数を使用して、`col_map` 列の各行の要素数を取得できます。

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
