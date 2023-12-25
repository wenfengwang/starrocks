---
displayed_sidebar: Chinese
---


# hll_union

## 機能

一連のHLL値の和集合を返します。

## 文法

```Haskell
hll_union(hll)
```

## パラメータ説明

`hll`: 他の列やインポートされたデータから生成されたHLL列です。

## 戻り値の説明

戻り値はHLL型です。

## 例

```Plain
mysql> select k1, hll_cardinality(hll_union(v1)) from tbl group by k1;
+------+----------------------------------+
| k1   | hll_cardinality(hll_union(`v1`)) |
+------+----------------------------------+
|    2 |                                4 |
|    1 |                                3 |
+------+----------------------------------+
```
