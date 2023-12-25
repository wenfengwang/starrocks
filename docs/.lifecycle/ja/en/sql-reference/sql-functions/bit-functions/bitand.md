---
displayed_sidebar: English
---

# bitand

## 説明

2つの数値式のビット単位のANDを返します。

## 構文

```Haskell
BITAND(x,y);
```

## パラメーター

- `x`: この式は、次のいずれかのデータ型に評価される必要があります: TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

- `y`: この式は、次のいずれかのデータ型に評価される必要があります: TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

> `x` と `y` はデータ型が一致している必要があります。

## 戻り値

戻り値の型は `x` と同じです。いずれかの値が NULL の場合、結果は NULL です。

## 例

```Plain Text
mysql> select bitand(3,0);
+--------------+
| bitand(3, 0) |
+--------------+
|            0 |
+--------------+
1行がセットされました (0.01 秒)
```
