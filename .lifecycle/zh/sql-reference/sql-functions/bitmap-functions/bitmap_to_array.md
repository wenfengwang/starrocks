---
displayed_sidebar: English
---

# 位图转换为数组

## 描述

将位图（BITMAP）转换成一个大整数（BIGINT）数组。

## 语法

```Haskell
 ARRAY<BIGINT> BITMAP_TO_ARRAY (bitmap)
```

## 参数

位图：需要转换的位图。

## 返回值

返回一个大整数（BIGINT）数组。

## 示例

```Plain
select bitmap_to_array(bitmap_from_string("1, 7"));
+----------------------------------------------+
| bitmap_to_array(bitmap_from_string('1, 7'))  |
+----------------------------------------------+
| [1,7]                                        |
+----------------------------------------------+

select bitmap_to_array(NULL);
+-----------------------+
| bitmap_to_array(NULL) |
+-----------------------+
| NULL                  |
+-----------------------+
```
