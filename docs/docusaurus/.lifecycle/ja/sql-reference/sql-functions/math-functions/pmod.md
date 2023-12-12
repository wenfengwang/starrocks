---
displayed_sidebar: "Japanese"
---

# pmod

## 説明

`dividend`を`divisor`で割ったときの正の余りを返します。

## 構文

```SQL
pmod(dividend, divisor)
```

## パラメーター

- `dividend`: 割られる数。
- `divisor`: 割る数。

`dividend`と`divisor`は以下のデータ型をサポートしています。

- BIGINT
- DOUBLE

> **注意**
>
> `dividend`と`divisor`のデータ型は一致している必要があります。StarRocksは、データ型が一致していない場合には暗黙的な変換を行います。

## 返り値

`dividend`と同じデータ型の値を返します。`divisor`が0で指定された場合、StarRocksはNULLを返します。

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