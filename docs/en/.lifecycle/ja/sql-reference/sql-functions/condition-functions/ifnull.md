---
displayed_sidebar: "Japanese"
---

# ifnull

## 説明

`expr1` が NULL の場合、`expr2` を返します。`expr1` が NULL でない場合、`expr1` を返します。

## 構文

```Haskell
ifnull(expr1,expr2);
```

## パラメーター

`expr1` と `expr2` はデータ型で互換性が必要です。

## 戻り値

戻り値のデータ型は `expr1` と同じです。

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
1 行が返されました (0.01 秒)
```
