---
displayed_sidebar: "Japanese"
---


# bitxor

## 説明

2つの数値式のビットごとの排他的論理和を返します。

## 構文

```Haskell
BITXOR(x,y);
```

## パラメーター

- `x`: この式は、次のデータ型のいずれかで評価する必要があります：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

- `y`: この式は、次のデータ型のいずれかで評価する必要があります：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

> `x` と `y` はデータ型で一致する必要があります。

## 戻り値

戻り値のデータ型は `x` と同じです。どちらかの値が NULL の場合、結果も NULL です。

## 例

```Plain Text
mysql> select bitxor(3,0);
+--------------+
| bitxor(3, 0) |
+--------------+
|            3 |
+--------------+
1 行が返されました (0.00 秒)
```
