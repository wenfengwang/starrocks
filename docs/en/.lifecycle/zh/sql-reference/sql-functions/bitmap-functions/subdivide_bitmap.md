---
displayed_sidebar: English
---

# subdivide_bitmap

## 描述

将大型位图分割成多个子位图。

此函数主要用于导出位图。过大的位图会超出MySQL协议允许的最大数据包大小。

该函数从v2.5版本开始支持。

## 语法

```Haskell
BITMAP subdivide_bitmap(bitmap, length)
```

## 参数

`bitmap`：需要被分割的位图，必填。
`length`：每个子位图的最大长度，必填。超过此值的位图会被分割成多个较小的位图。

## 返回值

返回多个不超过`length`的子位图。

## 示例

假设有一个表`t1`，其中`c2`列是BITMAP列。

```Plain
-- 使用bitmap_to_string()将`c2`中的值转换为字符串。
mysql> select c1, bitmap_to_string(c2) from t1;
+------+----------------------+
| c1   | bitmap_to_string(c2) |
+------+----------------------+
|    1 | 1,2,3,4,5,6,7,8,9,10 |
+------+----------------------+

-- 将`c2`分割成最大长度为3的小位图。

mysql> select c1, bitmap_to_string(subdivide_bitmap(c2, 3)) from t1;
+------+------------------------------------+
| c1   | bitmap_to_string(subdivide_bitmap) |
+------+------------------------------------+
|    1 | 1,2,3                              |
|    1 | 4,5,6                              |
|    1 | 7,8,9                              |
|    1 | 10                                 |
+------+------------------------------------+
```