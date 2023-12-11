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

## パラメーター

- `x`: この式は、次のデータ型のいずれかに評価される必要があります：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

- `y`: この式は、次のデータ型のいずれかに評価される必要があります：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

> `x` および `y` はデータ型で一致している必要があります。

## 戻り値

戻り値の型は `x` と同じです。いずれかの値がNULLの場合、結果もNULLになります。

## 例

```プレーンテキスト
mysql> select bitor(3,0);
+-------------+
| bitor(3, 0) |
+-------------+
|           3 |
+-------------+
1行が選択されました (0.00 秒)
```