---
displayed_sidebar: "Chinese"
---

# bitmap_to_string

## 描述

将输入的位图转换为由逗号(,)分隔的字符串。该字符串包含位图中的所有位。如果输入为null，则返回null。

## 语法

```Haskell
VARCHAR BITMAP_TO_STRING(BITMAP input)
```

## 参数

`input`: 要转换的位图。

## 返回值

返回一个VARCHAR类型的值。

## 示例

示例1：输入为null，返回null。

```Plain Text
MySQL > select bitmap_to_string(null);
+------------------------+
| bitmap_to_string(NULL) |
+------------------------+
| NULL                   |
+------------------------+
```

示例2：输入为空，返回一个空字符串。

```Plain Text
MySQL > select bitmap_to_string(bitmap_empty());
+----------------------------------+
| bitmap_to_string(bitmap_empty()) |
+----------------------------------+
|                                  |
+----------------------------------+
```

示例3：将包含一个位的位图转换为字符串。

```Plain Text
MySQL > select bitmap_to_string(to_bitmap(1));
+--------------------------------+
| bitmap_to_string(to_bitmap(1)) |
+--------------------------------+
| 1                              |
+--------------------------------+
```

示例4：将包含两个位的位图转换为字符串。

```Plain Text
MySQL > select bitmap_to_string(bitmap_or(to_bitmap(1), to_bitmap(2)));
+---------------------------------------------------------+
| bitmap_to_string(bitmap_or(to_bitmap(1), to_bitmap(2))) |
+---------------------------------------------------------+
| 1,2                                                     |
+---------------------------------------------------------+
```