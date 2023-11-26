---
displayed_sidebar: "Japanese"
---

# array_avg

## 説明

ARRAY内のすべてのデータの平均値を計算し、その結果を返します。

## 構文

```Haskell
array_avg(array(type))
```

`array(type)`は以下の要素のタイプをサポートしています：BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、DECIMALV2。

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

## キーワード

ARRAY_AVG,ARRAY
