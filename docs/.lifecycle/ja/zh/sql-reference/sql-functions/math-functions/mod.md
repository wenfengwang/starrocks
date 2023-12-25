---
displayed_sidebar: Chinese
---

# mod

## 機能

取模関数は、二つの数を除算した後の余りを返します。

以下の構文に従って、`dividend` を `divisor` で割った後の余りを返します。

## 構文

```Haskell
mod(dividend, divisor);
```

## パラメータ説明

- `dividend`: 被除数。サポートされるデータ型は TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、DECIMALV2、DECIMAL32、DECIMAL64、DECIMAL128です。
- `divisor`: 除数。サポートされるデータ型は `dividend` と同じです。

> 注：`dividend` と `divisor` のデータ型は一致している必要があります。一致しない場合は、暗黙の型変換が行われます。

## 戻り値の説明

戻り値のデータ型と符号は `dividend` と同じです。`divisor` が 0 の場合は、NULLを返します。

## 例

```Plain
mysql> select mod(3.14, 3.14);
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

select mod(11, -5);
+------------+
| mod(11, -5)|
+------------+
|          1 |
+------------+

select mod(-11, 5);
+-------------+
| mod(-11, 5) |
+-------------+
|          -1 |
+-------------+
```
