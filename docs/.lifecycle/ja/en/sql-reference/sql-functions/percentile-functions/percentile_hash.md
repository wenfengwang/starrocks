---
displayed_sidebar: English
---

# percentile_hash

## 説明

DOUBLE 値を PERCENTILE 値として構築します。

## 構文

```Haskell
PERCENTILE_HASH(x);
```

## パラメーター

`x`: サポートされるデータ型は DOUBLE です。

## 戻り値

PERCENTILE 値を返します。

## 例

```Plain Text
mysql> select percentile_approx_raw(percentile_hash(234.234), 0.99);
+-------------------------------------------------------+
| percentile_approx_raw(percentile_hash(234.234), 0.99) |
+-------------------------------------------------------+
|                                    234.23399353027344 |
+-------------------------------------------------------+
1 行がセットされました (0.00 秒)
```
