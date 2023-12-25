---
displayed_sidebar: Chinese
---

# nullif

## 機能

もし引数 `expr1` と `expr2` が等しい場合は NULL を返し、そうでなければ `expr1` の値を返します。

## 文法

```Haskell
nullif(expr1,expr2);
```

## 引数説明

`expr1` と `expr2` はデータ型が互換性がなければならず、そうでない場合はエラーを返します。

## 戻り値の説明

戻り値のデータ型は `expr1` の型と一致します。

## 例

```Plain Text
mysql> select nullif(1,2);
+--------------+
| nullif(1, 2) |
+--------------+
|            1 |
+--------------+

mysql> select nullif(1,1);
+--------------+
| nullif(1, 1) |
+--------------+
|         NULL |
+--------------+
1 row in set (0.01 sec)
```
