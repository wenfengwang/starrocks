---
displayed_sidebar: English
---

# bitmap_to_array

## 描述

将 BITMAP 转换成 BIGINT 数组。

## 语法

```Haskell
 ARRAY<BIGINT> BITMAP_TO_ARRAY (bitmap)
```

## 参数

`bitmap`：你想要转换的位图。

## 返回值

返回一个 BIGINT 数组。

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