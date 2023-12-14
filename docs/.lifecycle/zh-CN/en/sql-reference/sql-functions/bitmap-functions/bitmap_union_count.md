---
displayed_sidebar: "Chinese"
---

# bitmap_union_count

## 描述

返回一组位图值的并集，并返回并集的基数。该函数支持v2.3及以上版本。

## 语法

```Haskell
BIGINT bitmap_union_count(BITMAP value)
```

### 参数

`value`：一组位图值。支持的数据类型为BITMAP。

## 返回值

返回一个BIGINT类型的值。

## 示例

计算网页的唯一访问量（UV）。如果`user_id`是INT类型，则后两个查询是等价的。

```Plaintext
mysql> select * from test
+---------+---------+
| page_id | user_id |
+---------+---------+
|       1 |       1 |
|       1 |       2 |
|       2 |       1 |
+---------+---------+

mysql> select page_id,count(distinct user_id) from test group by page_id;
+---------+-------------------------+
| page_id | count(DISTINCT user_id) |
+---------+-------------------------+
|       1 |                       2 |
|       2 |                       1 |
+---------+-------------------------+

mysql> select page_id,bitmap_union_count(to_bitmap(user_id)) from test group by page_id;
+---------+----------------------------------------+
| page_id | bitmap_union_count(to_bitmap(user_id)) |
+---------+----------------------------------------+
|       1 |                                      2 |
|       2 |                                      1 |
+---------+----------------------------------------+
```