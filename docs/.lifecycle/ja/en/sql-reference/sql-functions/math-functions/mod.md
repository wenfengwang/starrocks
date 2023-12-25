---
displayed_sidebar: English
---

# mod

## 説明

`dividend` を `divisor` で割った際の剰余を返す関数です。

## 構文

```SQL
mod(dividend, divisor)
```

## パラメーター

- `dividend`: 除算される数値。
- `divisor`: 除算する数値。

`dividend` と `divisor` は以下のデータ型をサポートしています：

- TINYINT
- SMALLINT
- INT
- BIGINT
- LARGEINT
- FLOAT
- DOUBLE
- DECIMALV2
- DECIMAL32
- DECIMAL64
- DECIMAL128

> **注記**
>
> `dividend` と `divisor` は同じデータ型である必要があります。型が異なる場合、StarRocksは暗黙の型変換を行います。

## 戻り値

`dividend` と同じデータ型の値を返します。`divisor` が0として指定された場合、StarRocksはNULLを返します。

## 例

```Plain
mysql> select mod(3.14,3.14);
+-----------------+
| mod(3.14, 3.14) |
+-----------------+
|               0 |
+-----------------+

mysql> select mod(3.14, 3);
+--------------+
| mod(3.14, 3) |
+--------------+
|         0.14 |
+--------------+

select mod(11,-5);
+------------+
| mod(11, -5)|
+------------+
|          1 |
+------------+

select mod(-11,5);
+-------------+
| mod(-11, 5) |
+-------------+
|          -1 |
+-------------+
```
