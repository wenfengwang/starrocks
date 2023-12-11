---
displayed_sidebar: "Japanese"
---

# bitxor（ビット排他的論理和）

## 説明

2つの数値式の排他的論理和を返します。

## 構文

```Haskell
BITXOR(x,y);
```

## パラメーター

- `x`: この式は以下のいずれかのデータ型に評価される必要があります：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

- `y`: この式は以下のいずれかのデータ型に評価される必要があります：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

> `x` および `y` はデータ型で一致する必要があります。

## 戻り値

戻り値は `x` と同じ型になります。いずれかの値がNULLの場合、結果もNULLになります。

## 例

```Plain Text
mysql> select bitxor(3,0);
+--------------+
| bitxor(3, 0) |
+--------------+
|            3 |
+--------------+
1 row in set (0.00 sec)
```