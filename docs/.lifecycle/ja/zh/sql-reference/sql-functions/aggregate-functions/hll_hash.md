---
displayed_sidebar: Chinese
---

# hll_hash

## 機能

数値をHLL型に変換します。通常、データのインポート時に、ソースデータの数値をStarrocksのテーブルのHLL列にマッピングするために使用されます。

## 文法

```Haskell
HLL_HASH(column_name)
```

## パラメータ説明

`column_name`: 新しく生成されるHLL列の名前です。

## 戻り値の説明

HLL型の値を返します。

## 例

```plain text
mysql> select hll_cardinality(hll_hash("a"));
+--------------------------------+
| hll_cardinality(hll_hash('a')) |
+--------------------------------+
|                              1 |
+--------------------------------+
```
