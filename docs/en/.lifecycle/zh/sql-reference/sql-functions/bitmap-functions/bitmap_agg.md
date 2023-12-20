---
displayed_sidebar: English
---

# bitmap_agg

## 描述

将列中的值（排除 NULL）聚合成位图（多行合并为一行）。

## 语法

```Haskell
BITMAP_AGG(col)
```

## 参数

`col`：你想要聚合的列的值。它必须计算为 BOOLEAN、TINYINT、SMALLINT、INT、BIGINT 或 LARGEINT 类型。

## 返回值

返回 BITMAP 类型的值。

## 使用说明

如果某行的值小于 0 或大于 18446744073709551615，这个值将被忽略，不会被加入到位图中（参见示例 3）。

## 示例

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

SELECT * FROM t1_test ORDER BY c1;
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

示例 1：将 `c1` 列的值聚合成一个位图。

```PlainText
mysql> SELECT bitmap_to_string(bitmap_agg(c1)) FROM t1_test;
+----------------------------------+
| bitmap_to_string(bitmap_agg(c1)) |
+----------------------------------+
| 1,2,3,4,5,6                      |
+----------------------------------+
```

示例 2：将 `c2` 列的值聚合成一个位图（忽略 NULL）。

```PlainText
mysql> SELECT BITMAP_TO_STRING(BITMAP_AGG(c2)) FROM t1_test;
+----------------------------------+
| bitmap_to_string(bitmap_agg(c2)) |
+----------------------------------+
| 0,1                              |
+----------------------------------+
```

示例 3：将 `c6` 列的值聚合成一个位图（忽略超出值范围的最后两个值）。

```PlainText
mysql> SELECT bitmap_to_string(bitmap_agg(c6)) FROM t1_test;
+----------------------------------+
| bitmap_to_string(bitmap_agg(c6)) |
+----------------------------------+
| 11111,22222,33333                |
+----------------------------------+
```

## 关键词

BITMAP_AGG, BITMAP