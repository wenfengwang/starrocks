---
displayed_sidebar: "Japanese"
---

# map_values

## 説明

指定されたマップ内のすべての値の配列を返します。

この関数はv2.5からサポートされています。

## 構文

```Haskell
map_values(any_map)
```

## パラメータ

`any_map`: 値を取得したいMAPの値です。

## 戻り値

戻り値は`array<valueType>`の形式です。配列内の要素の型はマップ内の値の型と一致します。

入力がNULLの場合、NULLが返されます。MAPの値のキーまたは値がNULLの場合、NULLは通常の値として処理され、結果に含まれます。

## 例

### StarRocksネイティブテーブルからMAPデータをクエリする

v3.1以降、StarRocksではテーブルを作成する際にMAP列を定義することがサポートされています。この例では、次のデータを含む`test_map`というテーブルを使用します。

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

`col_map`列の各行からすべての値を取得します。

```SQL
select map_values(col_map) from test_map order by col_int;
+---------------------+
| map_values(col_map) |
+---------------------+
| [1,2]               |
| [3]                 |
| [4,5]               |
+---------------------+
3 rows in set (0.04 sec)
```

### データレイクからMAPデータをクエリする

この例では、次のデータを含むHiveテーブル`hive_map`を使用します。

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

[Hiveカタログ](../../../data_source/catalog/hive_catalog.md#create-a-hive-catalog)がクラスターに作成された後、このカタログとmap_values()関数を使用して、`col_map`列の各行からすべての値を取得できます。

```SQL
select map_values(col_map) from hive_map order by col_int;
+---------------------+
| map_values(col_map) |
+---------------------+
| [1,2]               |
| [3]                 |
| [4,5]               |
+---------------------+
3 rows in set (0.04 sec)
```
