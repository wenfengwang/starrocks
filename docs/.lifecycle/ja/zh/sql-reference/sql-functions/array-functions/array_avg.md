---
displayed_sidebar: Chinese
---

# array_avg

## 機能

ARRAY内の全データの平均値を求め、その結果を返します。

## 文法

```Haskell
array_avg(array(type))
```

## パラメータ説明

`array(type)` の `type` は以下のタイプをサポートしています：BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、DECIMALV2。

## 例

```plain text
mysql> select array_avg([11, 11, 12]);
+-----------------------+
| array_avg([11,11,12]) |
+-----------------------+
| 11.333333333333334    |
+-----------------------+

mysql> select array_avg([11.33, 11.11, 12.324]);
+---------------------------------+
| array_avg([11.33,11.11,12.324]) |
+---------------------------------+
| 11.588                          |
+---------------------------------+
```
