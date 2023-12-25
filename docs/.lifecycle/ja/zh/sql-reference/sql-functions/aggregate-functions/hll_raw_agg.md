---
displayed_sidebar: Chinese
---

# hll_raw_agg

## 機能

この関数は集約関数であり、HLL型のフィールドを集約するために使用され、返されるのもHLL型です。

## 文法

```Haskell
hll_raw_agg(hll)
```

## パラメータ説明

`hll`: 他の列やインポートされたデータから生成されたHLL列。

## 戻り値の説明

戻り値はHLL型です。

## 例

```Plain
mysql> select k1, hll_cardinality(hll_raw_agg(v1)) from tbl group by k1;
+------+----------------------------------+
| k1   | hll_cardinality(hll_raw_agg(`v1`)) |
+------+----------------------------------+
|    2 |                                4 |
|    1 |                                3 |
+------+----------------------------------+
```
