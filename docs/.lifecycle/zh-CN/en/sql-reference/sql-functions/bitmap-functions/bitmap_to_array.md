---
displayed_sidebar: "Chinese"
---

# bitmap_to_array

## 描述

将位图转换为BIGINT数组。

## 语法

```Haskell
 ARRAY<BIGINT> BITMAP_TO_ARRAY (bitmap)
```

## 参数

`bitmap`: 您要转换的位图。

## 返回值

返回一个BIGINT数组。

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