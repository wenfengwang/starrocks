---
displayed_sidebar: "Japanese"
---

# nullif

## 説明

`expr1` が `expr2` と等しい場合、NULL を返します。それ以外の場合は、`expr1` を返します。

## 構文

```Haskell
nullif(expr1,expr2);
```

## パラメーター

`expr1` と `expr2` はデータ型で互換性が必要です。

## 戻り値

戻り値のデータ型は `expr1` と同じです。

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
```
