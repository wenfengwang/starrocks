---
displayed_sidebar: English
---

# if

## 説明

`expr1` が TRUE と評価される場合、`expr2` を返します。それ以外の場合は、`expr3` を返します。

## 構文

```Haskell
if(expr1, expr2, expr3);
```

## パラメーター

`expr1`: 条件です。BOOLEAN 値でなければなりません。

`expr2` と `expr3` はデータ型が互換性がある必要があります。

## 戻り値

戻り値の型は `expr2` と同じです。

## 例

```Plain Text
mysql> select if(true, 1, 2);
+----------------+
| if(TRUE, 1, 2) |
+----------------+
|              1 |
+----------------+

mysql> select if(false, 2.14, 2);
+--------------------+
| if(FALSE, 2.14, 2) |
+--------------------+
|               2.00 |
+--------------------+
```
