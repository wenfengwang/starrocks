---
displayed_sidebar: "Chinese"
---

# array_to_bitmap

## 描述

将数组转换为BITMAP值。此功能从v2.3开始支持。

## 语法

```Haskell
BITMAP array_to_bitmap(array)
```

## 参数

`array`: 数组中的元素可以是INT、TINYINT或SMALLINT类型。

## 返回值

返回一个BITMAP类型的值。

## 使用注意事项

- 如果输入数组中元素的数据类型无效，例如STRING或DECIMAL，则会返回错误。

- 如果输入一个空数组，则会返回一个空的BITMAP值。

- 如果输入`NULL`，则返回`NULL`。

## 示例

示例1：将数组转换为BITMAP值。此函数必须嵌套在`bitmap_to_array`中，因为无法显示BITMAP值。

```Plain
MySQL > select bitmap_to_array(array_to_bitmap([1,2,3]));
+-------------------------------------------+
| bitmap_to_array(array_to_bitmap([1,2,3])) |
+-------------------------------------------+
| [1,2,3]                                   |
+-------------------------------------------+
```

示例2：输入一个空数组，会返回一个空数组。

```Plain
MySQL > select bitmap_to_array(array_to_bitmap([]));
+--------------------------------------+
| bitmap_to_array(array_to_bitmap([])) |
+--------------------------------------+
| []                                   |
+--------------------------------------+
```

示例3：输入`NULL`，会返回`NULL`。

```Plain
MySQL > select array_to_bitmap(NULL);
+-----------------------+
| array_to_bitmap(NULL) |
+-----------------------+
| NULL                  |
+-----------------------+
```