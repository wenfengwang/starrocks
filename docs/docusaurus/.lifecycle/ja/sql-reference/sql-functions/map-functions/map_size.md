---
displayed_sidebar: "日本語"
---

# map_size

## 説明

MAPの要素数を返します。MAPは、キーと値のペアの無順序なコレクションであり、例えば`{"a":1, "b":2}`のようになります。1つのキーと値のペアが1つの要素です。例えば`{"a":1, "b":2}`には2つの要素が含まれています。

この関数には[cardinality()](cardinality.md)という別名があります。v2.5からサポートされています。

## 構文

```Haskell
INT map_size(any_map)
```

## パラメータ

`any_map`: 要素数を取得したいMAPの値です。

## 戻り値

INTタイプの値を返します。

入力がNULLの場合、NULLが返されます。

MAPの値のキーまたは値がNULLの場合、NULLは通常の値として処理されます。

## 例

### StarRocksネイティブテーブルからMAPデータをクエリする

v3.1以降、StarRocksはテーブルを作成する際にMAP列を定義することをサポートしています。この例では、次のデータを含む`test_map`テーブルを使用しています。

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

`col_map`列の各行の要素数を取得します。

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

この例では、次のデータを含むHiveテーブル`hive_map`を使用しています。

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

[Hiveカタログ](../../../data_source/catalog/hive_catalog.md#create-a-hive-catalog)がクラスターに作成された後、このカタログとmap_size()関数を使用して、`col_map`列の各行の要素数を取得できます。

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