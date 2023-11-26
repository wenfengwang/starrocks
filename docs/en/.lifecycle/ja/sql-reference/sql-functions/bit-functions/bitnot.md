---
displayed_sidebar: "Japanese"
---

# bitnot

## 説明

数値式のビットワイズ否定を返します。

## 構文

```Haskell
BITNOT(x);
```

## パラメータ

`x`: この式は、次のデータ型のいずれかに評価される必要があります：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

## 戻り値

戻り値は `x` と同じ型です。値が NULL の場合、結果も NULL です。

## 例

```Plain Text
mysql> select bitnot(3);
+-----------+
| bitnot(3) |
+-----------+
|        -4 |
+-----------+
```
