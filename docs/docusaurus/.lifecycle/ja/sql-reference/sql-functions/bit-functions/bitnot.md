---
displayed_sidebar: "English"
---

# bitnot

## 説明

数値式のビット否定を返します。

## 構文

```Haskell
BITNOT(x);
```

## パラメータ

`x`: この式は、次のデータ型のいずれかに評価する必要があります：TINYINT, SMALLINT, INT, BIGINT, LARGEINT。

## 戻り値

戻り値は `x` と同じタイプです。どちらかの値がNULLの場合、結果もNULLです。

## 例

```Plain Text
mysql> select bitnot(3);
+-----------+
| bitnot(3) |
+-----------+
|        -4 |
+-----------+
```