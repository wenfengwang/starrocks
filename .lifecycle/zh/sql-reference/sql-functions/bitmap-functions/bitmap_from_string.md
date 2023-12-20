---
displayed_sidebar: English
---

# 字符串转换为位图

## 描述

此功能将字符串转换为位图。字符串由一系列用逗号分隔的 UINT32 数字构成。例如，字符串“0,1,2”会被转换成一个位图，在该位图中，第0、1和2位被设置。如果输入字段无效，则会返回 NULL。

该函数在转换过程中会对输入字符串进行去重处理。它必须与其他函数配合使用，比如 [bitmap_to_string](bitmap_to_string.md)。

## 语法

```Haskell
BITMAP BITMAP_FROM_STRING(VARCHAR input)
```

## 示例

```Plain

-- The input is empty and an empty value is returned.

MySQL > select bitmap_to_string(bitmap_empty());
+----------------------------------+
| bitmap_to_string(bitmap_empty()) |
+----------------------------------+
|                                  |
+----------------------------------+

-- `0,1,2` is returned.

MySQL > select bitmap_to_string(bitmap_from_string("0, 1, 2"));
+-------------------------------------------------+
| bitmap_to_string(bitmap_from_string('0, 1, 2')) |
+-------------------------------------------------+
| 0,1,2                                           |
+-------------------------------------------------+

-- `-1` is an invalid input and NULL is returned.

MySQL > select bitmap_to_string(bitmap_from_string("-1, 0, 1, 2"));
+-----------------------------------+
| bitmap_from_string('-1, 0, 1, 2') |
+-----------------------------------+
| NULL                              |
+-----------------------------------+

-- The input string is deduplicated.

MySQL > select bitmap_to_string(bitmap_from_string("0, 1, 1"));
+-------------------------------------------------+
| bitmap_to_string(bitmap_from_string('0, 1, 1')) |
+-------------------------------------------------+
| 0,1                                             |
+-------------------------------------------------+
```

## 关键字

BITMAP_FROM_STRING，位图
