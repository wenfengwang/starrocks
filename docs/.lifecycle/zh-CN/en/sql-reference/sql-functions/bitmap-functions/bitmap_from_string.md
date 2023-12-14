---
displayed_sidebar: "Chinese"
---

# bitmap_from_string

## 描述

将一个字符串转换成位图（BITMAP）。该字符串由一组以逗号分隔的UINT32数字组成。例如，“0, 1, 2”字符串将被转换为一个位图，其中位0、1和2被设置。如果输入字段无效，将返回NULL。

该函数在转换过程中对输入字符串进行去重。必须与其他函数一起使用，例如 [bitmap_to_string](bitmap_to_string.md)。

## 语法

```Haskell
BITMAP BITMAP_FROM_STRING(VARCHAR input)
```

## 示例

```Plain Text

-- 输入为空，返回一个空值。

MySQL > select bitmap_to_string(bitmap_empty());
+----------------------------------+
| bitmap_to_string(bitmap_empty()) |
+----------------------------------+
|                                  |
+----------------------------------+

-- 返回“0,1,2”。

MySQL > select bitmap_to_string(bitmap_from_string("0, 1, 2"));
+-------------------------------------------------+
| bitmap_to_string(bitmap_from_string('0, 1, 2')) |
+-------------------------------------------------+
| 0,1,2                                           |
+-------------------------------------------------+

-- “-1”是无效的输入，返回NULL。

MySQL > select bitmap_to_string(bitmap_from_string("-1, 0, 1, 2"));
+-----------------------------------+
| bitmap_from_string('-1, 0, 1, 2') |
+-----------------------------------+
| NULL                              |
+-----------------------------------+

-- 输入字符串被去重。

MySQL > select bitmap_to_string(bitmap_from_string("0, 1, 1"));
+-------------------------------------------------+
| bitmap_to_string(bitmap_from_string('0, 1, 1')) |
+-------------------------------------------------+
| 0,1                                             |
+-------------------------------------------------+
```

## 关键词

BITMAP_FROM_STRING, BITMAP