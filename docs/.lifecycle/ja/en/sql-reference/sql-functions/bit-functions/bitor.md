---
displayed_sidebar: English
---

# BITOR

## 説明

2つの数値式のビット単位のORを返します。

## 構文

```Haskell
BITOR(x,y);
```

## パラメーター

- `x`: この式は、TINYINT、SMALLINT、INT、BIGINT、LARGEINTのいずれかのデータ型に評価される必要があります。

- `y`: この式は、TINYINT、SMALLINT、INT、BIGINT、LARGEINTのいずれかのデータ型に評価される必要があります。

> `x` と `y` はデータ型が一致している必要があります。

## 戻り値

戻り値の型は `x` と同じです。いずれかの値がNULLの場合、結果はNULLになります。

## 例

```Plain Text
mysql> select bitor(3,0);
+-------------+
| bitor(3, 0) |
+-------------+
|           3 |
+-------------+
1行がセットされました (0.00 秒)
```
