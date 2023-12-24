---
displayed_sidebar: English
---

# bitmap_from_string

## 描述

将字符串转换为 BITMAP。该字符串由一组用逗号分隔的 UINT32 数字组成。例如，字符串 "0、1、2" 将被转换为一个位图，其中设置了位 0、1 和 2。如果输入字段无效，则返回 NULL。

在转换期间，此函数会对输入字符串进行重复数据删除。它必须与其他函数一起使用，例如 [bitmap_to_string](bitmap_to_string.md)。

## 语法

```Haskell
BITMAP BITMAP_FROM_STRING(VARCHAR input)
```

## 例子

```Plain Text

-- 输入为空，将返回一个空值。

MySQL > select bitmap_to_string(bitmap_empty());
+----------------------------------+
| bitmap_to_string(bitmap_empty()) |
+----------------------------------+
|                                  |
+----------------------------------+

-- 返回 `0,1,2`。

MySQL > select bitmap_to_string(bitmap_from_string("0, 1, 2"));
+-------------------------------------------------+
| bitmap_to_string(bitmap_from_string('0, 1, 2')) |
+-------------------------------------------------+
| 0,1,2                                           |
+-------------------------------------------------+

-- `-1` 是无效输入，将返回 NULL。

MySQL > select bitmap_to_string(bitmap_from_string("-1, 0, 1, 2"));
+-----------------------------------+
| bitmap_from_string('-1, 0, 1, 2') |
+-----------------------------------+
| NULL                              |
+-----------------------------------+

-- 输入字符串已进行重复数据删除。

MySQL > select bitmap_to_string(bitmap_from_string("0, 1, 1"));
+-------------------------------------------------+
| bitmap_to_string(bitmap_from_string('0, 1, 1')) |
+-------------------------------------------------+
| 0,1                                             |
+-------------------------------------------------+
```

## 关键字

BITMAP_FROM_STRING，BITMAP