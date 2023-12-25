---
displayed_sidebar: English
---

# coalesce

## 説明

入力パラメーターの中で最初のNULL以外の式を返します。NULL以外の式が見つからない場合はNULLを返します。

## 構文

```Haskell
coalesce(expr1,...);
```

## パラメーター

`expr1`: 互換性のあるデータ型に評価される必要がある入力式です。

## 戻り値

戻り値の型は`expr1`と同じです。

## 例

```Plain Text
mysql> select coalesce(3,NULL,1,1);
+-------------------------+
| coalesce(3, NULL, 1, 1) |
+-------------------------+
|                       3 |
+-------------------------+
1 row in set (0.00 sec)
```
