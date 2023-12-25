---
displayed_sidebar: Chinese
---

# sign

## 機能

引数 `x` の符号を返します。

`x` が負の数、0、正の数の場合、それぞれ -1、0、1 を返します。

## 文法

```Haskell
SIGN(x);
```

## 引数説明

`x`: 対応するデータ型は DOUBLE です。

## 戻り値の説明

戻り値のデータ型は FLOAT です。

## 例

```Plain Text
mysql> select sign(3.14159);
+---------------+
| sign(3.14159) |
+---------------+
|             1 |
+---------------+
1 row in set (0.02 sec)
```
