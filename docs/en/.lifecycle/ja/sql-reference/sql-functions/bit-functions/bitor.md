---
displayed_sidebar: "Japanese"
---

# bitor

## 説明

2つの数値式のビットごとのORを返します。

## 構文

```Haskell
BITOR(x,y);
```

## パラメータ

- `x`: この式は、次のいずれかのデータ型に評価される必要があります：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

- `y`: この式は、次のいずれかのデータ型に評価される必要があります：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

> `x` と `y` はデータ型で一致する必要があります。

## 戻り値

戻り値の型は `x` と同じです。どちらかの値が NULL の場合、結果も NULL です。

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
