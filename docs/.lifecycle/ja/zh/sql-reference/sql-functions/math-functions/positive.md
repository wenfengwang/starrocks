---
displayed_sidebar: Chinese
---

# positive

## 機能

式 `x` の結果を返します。

## 文法

```Haskell
POSITIVE(x);
```

## パラメータ説明

`x`: サポートされるデータ型は BIGINT、DOUBLE、DECIMALV2、DECIMAL32、DECIMAL64、DECIMAL128 です。

## 戻り値の説明

戻り値のデータ型はパラメータ `x` の型と同じです。

## 例

```Plain Text
mysql> select positive(3);
+-------------+
| positive(3) |
+-------------+
|           3 |
+-------------+
1行がセットされました (0.01 秒)

mysql> select positive(cast(3.14 as decimalv2));
+--------------------------------------+
| positive(CAST(3.14 AS DECIMAL(9,0))) |
+--------------------------------------+
|                                 3.14 |
+--------------------------------------+
1行がセットされました (0.01 秒)
```
