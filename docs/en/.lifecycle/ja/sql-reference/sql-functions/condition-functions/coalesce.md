---
displayed_sidebar: "Japanese"
---

# coalesce

## 説明

入力パラメーターの中で最初の非NULL式を返します。非NULLの式が見つからない場合はNULLを返します。

## 構文

```Haskell
coalesce(expr1,...);
```

## パラメーター

`expr1`: 互換性のあるデータ型に評価される入力式。

## 戻り値

戻り値は`expr1`と同じ型です。

## 例

```Plain Text
mysql> select coalesce(3,NULL,1,1);
+-------------------------+
| coalesce(3, NULL, 1, 1) |
+-------------------------+
|                       3 |
+-------------------------+
1 行が返されました (0.00 秒)
```
