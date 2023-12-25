---
displayed_sidebar: Chinese
---

# atan

## 機能

`x` のアークタンジェント（単位はラジアン）を返します。`x` はDOUBLE型の数値です。

## 構文

```Haskell
ATAN(x);
```

## 引数の説明

`x`: 対応しているデータ型は DOUBLE です。

## 戻り値の説明

戻り値のデータ型は DOUBLE です。入力値が NULL の場合、NULL を返します。

## 例

```Plain Text
mysql> select atan(2.5);
+--------------------+
| atan(2.5)          |
+--------------------+
| 1.1902899496825317 |
+--------------------+
1 row in set (0.01 sec)
```
