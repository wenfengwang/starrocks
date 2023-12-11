---
displayed_sidebar: "Japanese"
---

# abs

## 説明

数値 `x` の絶対値を返します。入力値がNULLの場合、NULLが返されます。

## 構文

```Haskell
ABS(x);
```

## パラメーター

`x`: 数値の値または式。

サポートされているデータ型: DOUBLE, FLOAT, LARGEINT, BIGINT, INT, SMALLINT, TINYINT, DECIMALV2, DECIMAL32, DECIMAL64, DECIMAL128。

## 戻り値

戻り値のデータ型は `x` の型と同じです。

## 例

```Plain Text
mysql> select abs(-1);
+---------+
| abs(-1) |
+---------+
|       1 |
+---------+
1 行が返されました (0.00 秒)
```

## キーワード

abs, absolute