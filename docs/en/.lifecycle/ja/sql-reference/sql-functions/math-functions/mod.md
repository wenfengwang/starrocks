---
displayed_sidebar: "Japanese"
---

# mod

## 説明

`dividend`を`divisor`で割った余りを返す剰余関数です。

## 構文

```SQL
mod(dividend, divisor)
```

## パラメータ

- `dividend`: 割られる数です。
- `divisor`: 割る数です。

`dividend`と`divisor`は以下のデータ型をサポートしています：

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

> **注意**
>
> `dividend`と`divisor`はデータ型で一致する必要があります。StarRocksはデータ型が一致しない場合、暗黙の型変換を行います。

## 戻り値

`dividend`と同じデータ型の値を返します。`divisor`が0で指定された場合、StarRocksはNULLを返します。

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
