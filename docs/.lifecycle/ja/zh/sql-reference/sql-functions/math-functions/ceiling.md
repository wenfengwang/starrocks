---
displayed_sidebar: Chinese
---

# CEILING

## 機能

`x` 以上の最小の整数を返します。

## 文法

```Haskell
CEILING(x);
```

## 引数説明

`x`: 対応するデータ型は DOUBLE です。

## 戻り値の説明

戻り値のデータ型は BIGINT です。

## 例

```Plain Text
mysql> select ceiling(3.14);
+---------------+
| ceiling(3.14) |
+---------------+
|             4 |
+---------------+
1 row in set (0.00 sec)
```
