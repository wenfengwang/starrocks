---
displayed_sidebar: English
---

# bitmap_to_array

## 描述

将 BITMAP 转换为 BIGINT 数组。

## 语法

```Haskell
 BITMAP_TO_ARRAY (bitmap) RETURNS ARRAY<BIGINT>
```

## 参数

`bitmap`: 要转换的位图。

## 返回值

返回一个 BIGINT 数组。

## 例子

```Plain
SELECT bitmap_to_array(bitmap_from_string("1, 7"));
+----------------------------------------------+
| bitmap_to_array(bitmap_from_string('1, 7'))  |
+----------------------------------------------+
| [1,7]                                        |
+----------------------------------------------+

SELECT bitmap_to_array(NULL);
+-----------------------+
| bitmap_to_array(NULL) |
+-----------------------+
| NULL                  |
+-----------------------+