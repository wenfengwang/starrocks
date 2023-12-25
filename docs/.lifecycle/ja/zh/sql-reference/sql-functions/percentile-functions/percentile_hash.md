---
displayed_sidebar: Chinese
---

# percentile_hash

## 機能

`double` 型の数値を `percentile` 型の数値に構築します。

## 文法

```Haskell
PERCENTILE_HASH(x);
```

## パラメータ説明

`x`: サポートされるデータ型は DOUBLE です。

## 戻り値の説明

戻り値のデータ型は PERCENTILE です。

## 例

```Plain Text
mysql> select percentile_approx_raw(percentile_hash(234.234), 0.99);
+-------------------------------------------------------+
| percentile_approx_raw(percentile_hash(234.234), 0.99) |
+-------------------------------------------------------+
|                                    234.23399353027344 |
+-------------------------------------------------------+
1 row in set (0.00 sec)
```
