---
displayed_sidebar: Chinese
---

# ifnull

## 機能

`expr1` が NULL でなければ `expr1` を返し、`expr1` が NULL であれば `expr2` を返します。

## 文法

```Haskell
ifnull(expr1,expr2);
```

## 引数説明

`expr1` と `expr2` はデータ型が互換性がなければならず、そうでない場合はエラーを返します。

## 戻り値の説明

戻り値のデータ型は `expr1` の型と一致します。

## 例

```Plain Text
mysql> select ifnull(2,4);
+--------------+
| ifnull(2, 4) |
+--------------+
|            2 |
+--------------+

mysql> select ifnull(NULL,2);
+-----------------+
| ifnull(NULL, 2) |
+-----------------+
|               2 |
+-----------------+
```
