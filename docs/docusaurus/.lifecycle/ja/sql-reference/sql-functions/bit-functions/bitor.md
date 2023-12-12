---
displayed_sidebar: "Japanese"
---

# bitor（ビット単位のOR）

## 説明

2つの数値式のビット単位のORを返します。

## 構文

```Haskell
BITOR(x,y);
```

## パラメーター

- `x`: この式は次のいずれかのデータ型に評価する必要があります：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

- `y`: この式は次のいずれかのデータ型に評価する必要があります：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

> `x` と `y` はデータ型で一致する必要があります。

## 戻り値

戻り値は `x` と同じ型です。どちらかの値がNULLの場合、結果もNULLになります。

## 例

```Plain Text
mysql> select bitor(3,0);
+-------------+
| bitor(3, 0) |
+-------------+
|           3 |
+-------------+
1 行が返されました (0.00 秒)
```