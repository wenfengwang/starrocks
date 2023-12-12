---
displayed_sidebar: "Japanese"
---

# map_keys

## 説明

指定されたマップ内のすべてのキーの配列を返します。

この関数はv2.5からサポートされています。

## 構文

```Haskell
map_keys(any_map)
```

## パラメータ

`any_map`: キーを取得したいMAP値です。

## 戻り値

戻り値は`array<keyType>`形式です。配列内の要素タイプは、マップ内のキータイプに一致します。

入力がNULLの場合、NULLが返されます。MAP値内のキーまたは値がNULLの場合、NULLは通常の値として処理され、結果に含まれます。

## 例

### StarRocksネイティブテーブルからMAPデータをクエリする

v3.1以降、StarRocksはテーブル作成時にMAP列の定義をサポートしています。この例では、次のデータを含む`test_map`テーブルを使用しています。

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

`col_map`列の各行からすべてのキーを取得します。

```Plaintext
select map_keys(col_map) from test_map order by col_int;
+-------------------+
| map_keys(col_map) |
+-------------------+
| ["a","b"]         |
| ["c"]             |
| ["d","e"]         |
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

[Hiveカタログ](../../../data_source/catalog/hive_catalog.md#create-a-hive-catalog)がクラスターに作成されたら、このカタログとmap_keys()関数を使用して`col_map`列の各行からすべてのキーを取得できます。

```Plaintext
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