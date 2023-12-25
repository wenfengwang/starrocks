---
displayed_sidebar: English
---

# hll_raw_agg

## 説明

この関数はHLLフィールドを集約するために使用される集約関数です。HLL値を返します。

## 構文

```Haskell
hll_raw_agg(hll)
```

## パラメーター

`hll`: 他のカラムから生成されるか、またはロードされたデータに基づいて生成されるHLLカラム。

## 戻り値

HLL型の値を返します。

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
