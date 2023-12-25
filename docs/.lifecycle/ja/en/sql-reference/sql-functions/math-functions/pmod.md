---
displayed_sidebar: English
---

# pmod

## 説明

`dividend` を `divisor` で割った際の正の余りを返します。

## 構文

```SQL
pmod(dividend, divisor)
```

## パラメーター

- `dividend`: 除算される数。
- `divisor`: 除算する数。

`dividend` と `divisor` は以下のデータ型をサポートしています:

- BIGINT
- DOUBLE

> **注記**
>
> `dividend` と `divisor` は同じデータ型である必要があります。データ型が異なる場合、StarRocksは暗黙の型変換を行います。

## 戻り値

`dividend` と同じデータ型の値を返します。`divisor` が0として指定された場合、StarRocksはNULLを返します。

## 例

```Plain
mysql> select pmod(3.14,3.14);
+------------------+
| pmod(3.14, 3.14) |
+------------------+
|                0 |
+------------------+

mysql> select pmod(3,6);
+------------+
| pmod(3, 6) |
+------------+
|          3 |
+------------+

mysql> select pmod(11,5);
+-------------+
| pmod(11, 5) |
+-------------+
|           1 |
+-------------+

mysql> select pmod(-11,5);
+--------------+
| pmod(-11, 5) |
+--------------+
|            4 |
+--------------+

mysql> SELECT pmod(11,-5);
+--------------+
| pmod(11, -5) |
+--------------+
|           -4 |
+--------------+

mysql> SELECT pmod(-11,-5);
+---------------+
| pmod(-11, -5) |
+---------------+
|            -1 |
+---------------+
```
