---
displayed_sidebar: English
---

# log2

## 説明

数値の2を底とする対数を計算します。

## 構文

```SQL
log2(arg)
```

## パラメーター

- `arg`: 対数を計算したい値です。DOUBLEデータ型のみサポートされています。

> **注記**
>
> StarRocksは、`arg`が負の数または0として指定された場合、NULLを返します。

## 戻り値

DOUBLEデータ型の値を返します。

## 例

例1: 8の2を底とする対数を計算します。

```Plain
mysql> select log2(8);
+---------+
| log2(8) |
+---------+
|       3 |
+---------+
1 row in set (0.00 sec)
```
