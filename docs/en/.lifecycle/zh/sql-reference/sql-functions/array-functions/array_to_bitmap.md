---
displayed_sidebar: English
---

# array_to_bitmap

## 描述

将数组转换为 BITMAP 值。该函数从 v2.3 版本开始支持。

## 语法

```Haskell
BITMAP array_to_bitmap(array)
```

## 参数

`array`：数组中的元素可以是 INT、TINYINT 或 SMALLINT 类型。

## 返回值

返回一个 BITMAP 类型的值。

## 使用说明

- 如果输入数组中的元素数据类型无效，比如 STRING 或 DECIMAL，会返回错误。

- 如果输入的是空数组，将返回一个空的 BITMAP 值。

- 如果输入的是 `NULL`，则返回 `NULL`。

## 示例

示例 1：将数组转换为 BITMAP 值。由于 BITMAP 值无法直接显示，此函数必须嵌套在 `bitmap_to_array` 函数中使用。

```Plain
MySQL > select bitmap_to_array(array_to_bitmap([1,2,3]));
+-------------------------------------------+
| bitmap_to_array(array_to_bitmap([1,2,3])) |
+-------------------------------------------+
| [1,2,3]                                   |
+-------------------------------------------+
```

示例 2：输入一个空数组，返回一个空数组。

```Plain
MySQL > select bitmap_to_array(array_to_bitmap([]));
+--------------------------------------+
| bitmap_to_array(array_to_bitmap([])) |
+--------------------------------------+
| []                                   |
+--------------------------------------+
```

示例 3：输入 `NULL`，返回 `NULL`。

```Plain
MySQL > select array_to_bitmap(NULL);
+-----------------------+
| array_to_bitmap(NULL) |
+-----------------------+
| NULL                  |
+-----------------------+
```