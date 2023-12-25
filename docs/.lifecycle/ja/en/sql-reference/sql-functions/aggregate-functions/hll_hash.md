---
displayed_sidebar: English
---

# hll_hash

## 説明

値をHLL型に変換します。通常、ソースデータの値をStarRocksテーブルのHLL列タイプにマッピングするために、インポート時に使用されます。

## 構文

```Haskell
HLL_HASH(column_name)
```

## パラメーター

`column_name`: 生成されるHLL列の名前です。

## 戻り値

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
