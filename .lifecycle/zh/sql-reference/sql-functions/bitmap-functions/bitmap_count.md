---
displayed_sidebar: English
---

# 位图计数

## 描述

返回输入位图中1的个数。

## 语法

```Haskell
INT BITMAP_COUNT(any_bitmap)
```

## 示例

```Plain
MySQL > select bitmap_count(bitmap_from_string("1,2,4"));
+-------------------------------------------+
| bitmap_count(bitmap_from_string('1,2,4')) |
+-------------------------------------------+
|                                         3 |
+-------------------------------------------+

MySQL > select bitmap_count(NULL);
+--------------------+
| bitmap_count(NULL) |
+--------------------+
|                  0 |
+--------------------+
```

## 关键字

位图，位图计数
