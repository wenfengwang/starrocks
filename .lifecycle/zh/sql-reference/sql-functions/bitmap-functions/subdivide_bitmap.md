---
displayed_sidebar: English
---

# 位图细分

## 描述

此功能将一个大型位图分割成多个子位图。

此函数主要用于导出位图。如果位图文件过大，会超出MySQL协议所允许的最大数据包尺寸。

此功能自v2.5版本起提供支持。

## 语法

```Haskell
BITMAP subdivide_bitmap(bitmap, length)
```

## 参数

bitmap：需要被分割的位图，此参数为必需。length：每个子位图的最大长度，此参数为必需。超过此长度的位图会被分割成若干个较小的位图。

## 返回值

返回若干个长度不超过设定值的子位图。

## 示例

比如有一个名为t1的表，其中的c2列是一个BITMAP类型的列。

```Plain
-- Use bitmap_to_string() to convert values in `c2` into a string.
mysql> select c1, bitmap_to_string(c2) from t1;
+------+----------------------+
| c1   | bitmap_to_string(c2) |
+------+----------------------+
|    1 | 1,2,3,4,5,6,7,8,9,10 |
+------+----------------------+

-- Split `c2` into small bitmaps whose maximum length is 3.

mysql> select c1, bitmap_to_string(subdivide_bitmap) from t1, subdivide_bitmap(c2, 3);
+------+------------------------------------+
| c1   | bitmap_to_string(subdivide_bitmap) |
+------+------------------------------------+
|    1 | 1,2,3                              |
|    1 | 4,5,6                              |
|    1 | 7,8,9                              |
|    1 | 10                                 |
+------+------------------------------------+
```
