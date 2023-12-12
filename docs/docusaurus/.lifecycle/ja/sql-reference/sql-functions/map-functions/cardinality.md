---
displayed_sidebar: "英語"
---

# カーディナリティ

## 説明

MAP値の要素数を返します。MAPはキーと値のペアの順序不定のコレクションで、例えば `{"a":1, "b":2}` のようです。1つのキーと値のペアが1つの要素を構成します。`{"a":1, "b":2}` には2つの要素が含まれています。

この関数はv3.0以降でサポートされています。[map_size()](map_size.md)の別名です。

## 構文

```Haskell
INT cardinality(any_map)
```

## パラメータ

`any_map`: 要素数を取得したいMAP値。

## 戻り値

INT値の値を返します。

入力がNULLの場合、NULLが返されます。

MAP値のキーまたは値がNULLの場合、NULLは通常の値として処理されます。

## 例

### StarRocksのネイティブテーブルからMAPデータをクエリする

v3.1以降、StarRocksはテーブルを作成するときにMAP列を定義することをサポートしています。この例では、以下のデータを含むテーブル `test_map` を使用しています：

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
3行セット (0.05 秒)
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
3行セット (0.05 秒)
```

### データレイクからのMAPデータをクエリする

この例では、以下のデータを含むHive表 `hive_map` を使用しています：

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

クラスターに [Hiveカタログ](../../../data_source/catalog/hive_catalog.md#create-a-hive-catalog) が作成された後、このカタログとcardinality()関数を使用して、`col_map`列の各行の要素数を取得することができます。

```Plaintext
SELECT cardinality(col_map) FROM hive_map ORDER BY col_int;
+----------------------+
| cardinality(col_map) |
+----------------------+
|                    2 |
|                    1 |
|                    2 |
+----------------------+
3行セット (0.05 秒)
```