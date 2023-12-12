```yaml
---
displayed_sidebar: "Japanese"
---


# bitxor（ビット排他的論理和）

## 説明

2つの数値式のビット排他的論理和を返します。

## 構文

```Haskell
BITXOR(x,y);
```

## パラメーター

- `x`: この式は、次のデータ型のいずれかに評価される必要があります：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

- `y`: この式は、次のデータ型のいずれかに評価される必要があります：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

> `x` と `y` はデータ型で同意する必要があります。

## 戻り値

戻り値の型は `x` と同じです。いずれかの値がNULLの場合、結果もNULLになります。

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