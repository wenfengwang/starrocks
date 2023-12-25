---
displayed_sidebar: English
---

# positive

## 説明

`x`の値をそのまま返します。

## 構文

```Haskell
POSITIVE(x);
```

## パラメーター

`x`: BIGINT、DOUBLE、DECIMALV2、DECIMAL32、DECIMAL64、およびDECIMAL128データ型に対応しています。

## 戻り値

`x`のデータ型と同じデータ型の値を返します。

## 例

```Plain
mysql> select positive(3);
+-------------+
| positive(3) |
+-------------+
|           3 |
+-------------+
1 row in set (0.01 sec)

mysql> select positive(cast(3.14 as DECIMALV2));
+--------------------------------------+
| positive(CAST(3.14 AS DECIMAL(9,0))) |
+--------------------------------------+
|                                 3.14 |
+--------------------------------------+
1 row in set (0.01 sec)
```
