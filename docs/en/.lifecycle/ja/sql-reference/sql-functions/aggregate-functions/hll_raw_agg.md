---
displayed_sidebar: "Japanese"
---

# hll_raw_agg

## 説明

この関数は、HLLフィールドを集計するために使用される集計関数です。HLLの値を返します。

## 構文

```Haskell
hll_raw_agg(hll)
```

## パラメータ

`hll`: 他の列によって生成されるか、ロードされたデータに基づいて生成されたHLL列です。

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
