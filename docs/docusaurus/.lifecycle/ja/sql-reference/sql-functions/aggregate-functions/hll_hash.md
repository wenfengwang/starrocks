---
displayed_sidebar: "Japanese"
---

# hll_hash

## 説明

値をhllタイプに変換します。通常、ソースデータの値をStarRocksテーブルのHLL列タイプにマップするためにインポートで使用されます。

## 構文

```Haskell
HLL_HASH(column_name)
```

## パラメーター

`column_name`: 生成されたHLL列の名前。

## 戻り値

HLLタイプの値を返します。

## 例

```plain text
mysql> select hll_cardinality(hll_hash("a"));
+--------------------------------+
| hll_cardinality(hll_hash('a')) |
+--------------------------------+
|                              1 |
+--------------------------------+
```