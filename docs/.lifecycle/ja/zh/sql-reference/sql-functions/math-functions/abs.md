---
displayed_sidebar: Chinese
---

# abs

## 機能

`x` の絶対値を返します。入力値が NULL の場合は、NULL を返します。

## 構文

```Haskell
ABS(x);
```

## 引数説明

`x`: 対応するデータ型は DOUBLE、FLOAT、LARGEINT、BIGINT、INT、SMALLINT、TINYINT、DECIMALV2、DECIMAL32、DECIMAL64、DECIMAL128 です。

## 戻り値の説明

戻り値のデータ型は `x` の型と同じです。

## 例

```Plain Text
mysql> select abs(-1);
+---------+
| abs(-1) |
+---------+
|       1 |
+---------+
1 row in set (0.00 sec)
```

## キーワード

abs, absolute
