---
displayed_sidebar: Chinese
---

# map_values

## 機能

Map内のすべてのValueを配列として返します。Mapにはキーと値のペア（key-value pair）が保存されています。例えば `{"a":1, "b":2}` のように。

この関数はバージョン2.5からサポートされています。

## 文法

```Haskell
map_values(any_map)
```

## 引数説明

`any_map`: Valuesを取得するMapの値で、Map型でなければなりません。

## 戻り値の説明

ARRAY型の配列を返し、形式は `array<valueType>` です。`valueType` のデータ型はMapのValueのデータ型と同じです。

入力パラメータがNULLの場合、結果もNULLになります。

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

`col_map` 列の各行のすべてのvaluesを取得します。

```SQL
SELECT map_values(col_map) FROM test_map ORDER BY col_int;
+---------------------+
| map_values(col_map) |
+---------------------+
| [1,2]               |
| [3]                 |
| [4,5]               |
+---------------------+
3 rows in set (0.04 sec)
```

### 外部データレイクでのMapデータのクエリ

Hiveに `hive_map` というテーブルがあると仮定し、そのデータは以下の通りです：

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

StarRocksクラスタで[Hiveカタログを作成する](../../../data_source/catalog/hive_catalog.md#hive-カタログの作成)ことにより、このテーブルにアクセスし、`col_map` 列の各行のすべてのvaluesを取得します。

```SQL
SELECT map_values(col_map) FROM hive_map ORDER BY col_int;
+---------------------+
| map_values(col_map) |
+---------------------+
| [1,2]               |
| [3]                 |
| [4,5]               |
+---------------------+
3 rows in set (0.04 sec)
```
