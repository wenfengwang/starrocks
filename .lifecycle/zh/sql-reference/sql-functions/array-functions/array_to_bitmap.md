---
displayed_sidebar: English
---

# 数组转换为位图

## 描述

将数组转换成位图（BITMAP）值。此函数从v2.3版本开始支持。

## 语法

```Haskell
BITMAP array_to_bitmap(array)
```

## 参数

array：数组中的元素可以是 INT、TINYINT 或 SMALLINT 类型。

## 返回值

返回位图（BITMAP）类型的值。

## 使用须知

- 如果输入数组中的元素数据类型不合法，如 STRING 或 DECIMAL，会返回错误。

- 如果输入的是空数组，将返回一个空的位图（BITMAP）值。

- 如果输入的是 NULL，将返回 NULL。

## 示例

示例 1：将一个数组转换为位图（BITMAP）值。这个函数必须嵌套在 bitmap_to_array 函数中使用，因为位图（BITMAP）值无法直接显示。

```Plain
MySQL > select bitmap_to_array(array_to_bitmap([1,2,3]));
+-------------------------------------------+
| bitmap_to_array(array_to_bitmap([1,2,3])) |
+-------------------------------------------+
| [1,2,3]                                   |
+-------------------------------------------+
```

示例 2：输入一个空数组，将返回一个空数组。

```Plain
MySQL > select bitmap_to_array(array_to_bitmap([]));
+--------------------------------------+
| bitmap_to_array(array_to_bitmap([])) |
+--------------------------------------+
| []                                   |
+--------------------------------------+
```

示例 3：输入 NULL，将返回 NULL。

```Plain
MySQL > select array_to_bitmap(NULL);
+-----------------------+
| array_to_bitmap(NULL) |
+-----------------------+
| NULL                  |
+-----------------------+
```
