---
displayed_sidebar: Chinese
---

# map_keys

## 機能

Map 中のすべての key を配列として返します。Map にはキーと値のペア（key-value pair）が保存されています。例えば `{"a":1, "b":2}` のように。

この関数はバージョン 2.5 からサポートされています。

## 文法

```Haskell
ARRAY map_keys(any_map)
```

## パラメータ説明

`any_map`: キーを取得する Map の値。

## 戻り値の説明

`array<keyType>` 形式の ARRAY 型の配列を返します。`keyType` は Map のキーのデータ型と同じです。

入力パラメータが NULL の場合、結果も NULL です。Map のキーまたは値が NULL の場合、その NULL 値は正常に返されます。

## 例

### StarRocks のローカルテーブルでの Map データのクエリ

バージョン 3.1 では、テーブル作成時に Map 型の列を定義することがサポートされています。`test_map` テーブルを作成する例を以下に示します。

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

`col_map` 列の各行のすべてのキーを取得します。

```Plain
SELECT map_keys(col_map) FROM test_map ORDER BY col_int;
+-------------------+
| map_keys(col_map) |
+-------------------+
| ["a","b"]         |
| ["c"]             |
| ["d","e"]         |
+-------------------+
```

### 外部データレイクでの Map データのクエリ

Hive に `hive_map` というテーブルがあり、そのデータは以下の通りです：

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

StarRocks クラスタで [Hive catalog を作成する](../../../data_source/catalog/hive_catalog.md#hive-catalogの作成) ことにより、このテーブルにアクセスし、`col_map` 列の各行のすべてのキーを取得します。

```Plain
select map_keys(col_map) from hive_map order by col_int;
+-------------------+
| map_keys(col_map) |
+-------------------+
| ["a","b"]         |
| ["c"]             |
| ["d","e"]         |
+-------------------+
3 rows in set (0.05 sec)
```
