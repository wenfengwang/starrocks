---
displayed_sidebar: English
---

# bitmap_agg

## 描述

将列中的值（不包括 NULL）聚合到位图中（将多行聚合为一行）。

## 语法

```Haskell
BITMAP_AGG(col)
```

## 参数

`col`：要进行值聚合的列。其计算结果必须为 BOOLEAN、TINYINT、SMALLINT、INT、BIGINT 和 LARGEINT。

## 返回值

返回一个 BITMAP 类型的值。

## 使用说明

如果行中的值小于 0 或大于 18446744073709551615，则该值将被忽略，不会被添加到位图中（参见示例 3）。

## 例子

以以下数据表为例：

```PlainText
mysql> CREATE TABLE t1_test (
    c1 int,
    c2 boolean,
    c3 tinyint,
    c4 int,
    c5 bigint,
    c6 largeint
    )
DUPLICATE KEY(c1)
DISTRIBUTED BY HASH(c1)
BUCKETS 1
PROPERTIES ("replication_num" = "3");

INSERT INTO t1_test VALUES
    (1, true, 11, 111, 1111, 11111),
    (2, false, 22, 222, 2222, 22222),
    (3, true, 33, 333, 3333, 33333),
    (4, null, null, null, null, null),
    (5, -1, -11, -111, -1111, -11111),
    (6, null, null, null, null, "36893488147419103232");

select * from t1_test order by c1;
+------+------+------+------+-------+----------------------+
| c1   | c2   | c3   | c4   | c5    | c6                   |
+------+------+------+------+-------+----------------------+
|    1 |    1 |   11 |  111 |  1111 | 11111                |
|    2 |    0 |   22 |  222 |  2222 | 22222                |
|    3 |    1 |   33 |  333 |  3333 | 33333                |
|    4 | NULL | NULL | NULL |  NULL | NULL                 |
|    5 |    1 |  -11 | -111 | -1111 | -11111               |
|    6 | NULL | NULL | NULL |  NULL | 36893488147419103232 |
+------+------+------+------+-------+----------------------+
```

示例 1：将列 `c1` 中的值聚合到一个位图中。

```PlainText
mysql> select bitmap_to_string(bitmap_agg(c1)) from t1_test;
+----------------------------------+
| bitmap_to_string(bitmap_agg(c1)) |
+----------------------------------+
| 1,2,3,4,5,6                      |
+----------------------------------+
```

示例 2：将列 `c2` 中的值聚合到一个位图中（忽略 NULL）。

```PlainText
mysql> SELECT BITMAP_TO_STRING(BITMAP_AGG(c2)) FROM t1_test;
+----------------------------------+
| bitmap_to_string(bitmap_agg(c2)) |
+----------------------------------+
| 0,1                              |
+----------------------------------+
```

示例 3：将列 `c6` 中的值聚合到一个位图中（忽略超出值范围的最后两个值）。

```PlainText
mysql> select bitmap_to_string(bitmap_agg(c6)) from t1_test;
+----------------------------------+
| bitmap_to_string(bitmap_agg(c6)) |
+----------------------------------+
| 11111,22222,33333                |
+----------------------------------+
```

## 关键字

BITMAP_AGG、BITMAP
