---
displayed_sidebar: "Japanese"
---

# coalesce（翻訳）

## 説明

入力パラメーターの中で最初の非NULL式を返します。非NULLの式が見つからない場合は、NULLを返します。

## 構文

```Haskell
coalesce(expr1,...);
```

## パラメーター

`expr1`: 互換性のあるデータ型に評価される必要がある入力式。

## 戻り値

戻り値の型は `expr1` と同じです。

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