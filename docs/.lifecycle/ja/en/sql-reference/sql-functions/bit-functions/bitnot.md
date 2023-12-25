---
displayed_sidebar: English
---

# bitnot

## 説明

数値式のビット単位の否定を返します。

## 構文

```Haskell
BITNOT(x);
```

## パラメーター

`x`: この式は、TINYINT、SMALLINT、INT、BIGINT、LARGEINT のいずれかのデータ型に評価される必要があります。

## 戻り値

戻り値の型は `x` と同じです。いずれかの値が NULL の場合、結果は NULL になります。

## 例

```Plain Text
mysql> select bitnot(3);
+-----------+
| bitnot(3) |
+-----------+
|        -4 |
+-----------+
```
