---
displayed_sidebar: Chinese
---

# hll_cardinality

## 機能

HLL 型の値の基数を計算するために使用されます。

## 文法

```Haskell
HLL_CARDINALITY(hll)
```

## パラメータ説明

`hll`: 他の列やインポートされたデータから生成された HLL 列。

## 戻り値の説明

BIGINT 型の値を返します。

## 例

```plain text
MySQL > select HLL_CARDINALITY(uv_set) from test_uv;
+---------------------------+
| hll_cardinality(`uv_set`) |
+---------------------------+
|                         3 |
+---------------------------+
```
