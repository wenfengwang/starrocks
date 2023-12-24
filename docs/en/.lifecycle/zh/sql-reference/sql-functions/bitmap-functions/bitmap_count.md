---
displayed_sidebar: English
---

# bitmap_count

## 描述

返回输入位图的 1 位计数。

## 语法

```Haskell
INT BITMAP_COUNT(any_bitmap)
```

## 例子

```Plain Text
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

## 关键词

BITMAP，BITMAP_COUNT