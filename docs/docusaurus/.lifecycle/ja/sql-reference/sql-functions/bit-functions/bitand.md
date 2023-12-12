---
displayed_sidebar: "Japanese"
---

# bitand

## 説明

2つの数値式のビットごとのANDを返します。

## 構文

```Haskell
BITAND(x,y);
```

## パラメーター

- `x`: この式は次のデータ型のいずれかに評価する必要があります：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

- `y`: この式は次のデータ型のいずれかに評価する必要があります：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

> `x` と `y` はデータ型で同意する必要があります。

## 戻り値

戻り値は `x` と同じタイプです。いずれかの値がNULLの場合、結果もNULLです。

## 例

```Plain Text
mysql> select bitand(3,0);
+--------------+
| bitand(3, 0) |
+--------------+
|            0 |
+--------------+
1 row in set (0.01 sec)
```