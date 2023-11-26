---
displayed_sidebar: "Japanese"
---

# percentile_hash

## 説明

PERCENTILE値としてDOUBLE値を構築します。

## 構文

```Haskell
PERCENTILE_HASH(x);
```

## パラメータ

`x`: サポートされるデータ型はDOUBLEです。

## 返り値

PERCENTILE値を返します。

## 例

```Plain Text
mysql> select percentile_approx_raw(percentile_hash(234.234), 0.99);
+-------------------------------------------------------+
| percentile_approx_raw(percentile_hash(234.234), 0.99) |
+-------------------------------------------------------+
|                                    234.23399353027344 |
+-------------------------------------------------------+
1 行が返されました (0.00 秒)
```
