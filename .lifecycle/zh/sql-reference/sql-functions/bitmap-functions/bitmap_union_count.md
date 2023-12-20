---
displayed_sidebar: English
---

# 位图合并计数

## 描述

返回一组位图值的合并结果，并返回该合并结果的基数。该函数自v2.3版本起提供支持。

## 语法

```Haskell
BIGINT bitmap_union_count(BITMAP value)
```

### 参数

value：一组位图值。支持的数据类型为 BITMAP。

## 返回值

返回类型为 BIGINT 的值。

## 示例

计算网页的独立访客数（UV）。如果 user_id 是 INT 类型，那么后两种查询方式是等效的。

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
