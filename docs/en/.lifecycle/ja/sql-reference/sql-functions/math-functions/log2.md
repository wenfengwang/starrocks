---
displayed_sidebar: "Japanese"
---

# log2

## 説明

数値の底が2の対数を計算します。

## 構文

```SQL
log2(arg)
```

## パラメータ

- `arg`: 対数を計算したい値。DOUBLEデータ型のみサポートされています。

> **注意**
>
> `arg`が負数または0で指定された場合、StarRocksはNULLを返します。

## 戻り値

DOUBLEデータ型の値を返します。

## 例

例1: 8の底が2の対数を計算する。

```Plain
mysql> select log2(8);
+---------+
| log2(8) |
+---------+
|       3 |
+---------+
1 row in set (0.00 sec)
```
