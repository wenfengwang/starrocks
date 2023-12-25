---
displayed_sidebar: Chinese
---

# conv

## 機能

入力されたパラメータ `x` の進数変換を行います。

## 文法

```Haskell
CONV(x,y,z);
```

## パラメータ説明

`x`: 対応するデータ型は VARCHAR、BIGINT です。

`y`: 対応するデータ型は TINYINT、元の進数です。

`z`: 対応するデータ型は TINYINT、変換後の進数です。

## 戻り値の説明

戻り値のデータ型は VARCHAR です。

## 例

```Plain Text
mysql> select conv(8,10,2);
+----------------+
| conv(8, 10, 2) |
+----------------+
| 1000           |
+----------------+
1 row in set (0.00 sec)
```
