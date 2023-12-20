---
displayed_sidebar: English
---

# bitmap_from_string

## 描述

将字符串转换为 BITMAP。该字符串由一组以逗号分隔的 UINT32 数字组成。例如，字符串 "0, 1, 2" 将被转换为一个 Bitmap，在该 Bitmap 中，位 0、1 和 2 被设置。如果输入字段无效，则返回 NULL。

此函数在转换过程中会对输入字符串进行去重。它必须与其他函数一起使用，例如 [bitmap_to_string](bitmap_to_string.md)。

## 语法

```Haskell
BITMAP BITMAP_FROM_STRING(VARCHAR input)
```

## 示例

```Plain

-- 输入为空，返回一个空值。

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

-- `-1` 是一个无效输入，返回 NULL。

MySQL > select bitmap_to_string(bitmap_from_string("-1, 0, 1, 2"));
+-----------------------------------+
| bitmap_from_string('-1, 0, 1, 2') |
+-----------------------------------+
| NULL                              |
+-----------------------------------+

-- 输入字符串进行了去重。

MySQL > select bitmap_to_string(bitmap_from_string("0, 1, 1"));
+-------------------------------------------------+
| bitmap_to_string(bitmap_from_string('0, 1, 1')) |
+-------------------------------------------------+
| 0,1                                             |
+-------------------------------------------------+
```

## 关键词

BITMAP_FROM_STRING，BITMAP