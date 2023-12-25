---
displayed_sidebar: English
---

# cardinality

## 説明

MAP値の要素数を返します。MAPは、キーと値のペアの順序付けされていないコレクションです（例: `{"a":1, "b":2}`）。1つのキーと値のペアが1つの要素を構成します。`{"a":1, "b":2}`には2つの要素が含まれています。

この関数はv3.0以降でサポートされています。これは[map_size()](map_size.md)のエイリアスです。

## 構文

```Haskell
INT cardinality(any_map)
```

## パラメータ

`any_map`: 要素数を取得したいMAP値。

## 戻り値

INT型の値を返します。

入力がNULLの場合、NULLが返されます。

MAP値のキーまたは値がNULLの場合、NULLは通常の値として処理されます。

## 例

### StarRocksネイティブテーブルからのMAPデータのクエリ

v3.1以降、StarRocksはテーブル作成時にMAP列を定義することをサポートしています。この例では、`test_map`テーブルを使用し、次のデータが含まれています：

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
select cardinality(col_map) from test_map order by col_int;
+----------------------+
| cardinality(col_map) |
+----------------------+
|                    2 |
|                    1 |
|                    2 |
+----------------------+
3 rows in set (0.05 sec)
```

### データレイクからのMAPデータのクエリ

この例では、次のデータを含むHiveテーブル`hive_map`を使用します：

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

クラスターに[Hiveカタログ](../../../data_source/catalog/hive_catalog.md#create-a-hive-catalog)を作成した後、このカタログとcardinality()関数を使用して、`col_map`列の各行の要素数を取得できます。

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
