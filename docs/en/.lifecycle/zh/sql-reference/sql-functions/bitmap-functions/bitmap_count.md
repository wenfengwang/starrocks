---
displayed_sidebar: English
---

# bitmap_count

## 描述

返回输入位图的1位计数。

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

BITMAP,BITMAP_COUNT