---
displayed_sidebar: English
---

# array_avg

## 描述

计算数组中所有数据的平均值并返回结果。

## 语法

```Haskell
array_avg(array(type))
```

`array(type)` 支持以下类型的元素：BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、DECIMALV2。

## 例子

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

## 关键词

ARRAY_AVG, ARRAY