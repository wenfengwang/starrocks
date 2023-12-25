---
displayed_sidebar: English
---

# map_size

## 説明

MAP 値の要素数を返します。MAP は、キーと値のペアの順序付けされていないコレクションで、例えば `{"a":1, "b":2}` のようになります。1 つのキーと値のペアは1つの要素を構成し、`{"a":1, "b":2}` は2つの要素を含んでいます。

この関数は [cardinality()](cardinality.md) という別名も持っており、v2.5 からサポートされています。

## 構文

```Haskell
INT map_size(any_map)
```

## パラメーター

`any_map`: 要素数を取得したい MAP 値。

## 戻り値

INT 型の値を返します。

入力が NULL の場合、NULL が返されます。

MAP 値のキーまたは値が NULL の場合、NULL は通常の値として処理されます。

## 例

### StarRocks ネイティブテーブルからの MAP データのクエリ

v3.1 以降、StarRocks はテーブル作成時に MAP 列を定義することをサポートしています。この例では `test_map` テーブルを使用し、以下のデータを含んでいます。

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

### データレイクからの MAP データのクエリ

この例では、以下のデータを含む Hive テーブル `hive_map` を使用します。

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

クラスターに [Hive カタログ](../../../data_source/catalog/hive_catalog.md#create-a-hive-catalog) を作成した後、このカタログと map_size() 関数を使用して `col_map` 列の各行の要素数を取得できます。

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
