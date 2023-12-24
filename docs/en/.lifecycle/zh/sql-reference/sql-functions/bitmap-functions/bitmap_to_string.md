---
displayed_sidebar: English
---

# bitmap_to_string

## 描述

将输入的位图转换为以逗号（,）分隔的字符串。该字符串包含位图中的所有位。如果输入为 null，则返回 null。

## 语法

```Haskell
VARCHAR BITMAP_TO_STRING(BITMAP input)
```

## 参数

`input`：要转换的位图。

## 返回值

返回 VARCHAR 类型的值。

## 例子

示例 1：输入为 null 时返回 null。

```Plain Text
MySQL > select bitmap_to_string(null);
+------------------------+
| bitmap_to_string(NULL) |
+------------------------+
| NULL                   |
+------------------------+
```

示例 2：输入为空时返回空字符串。

```Plain Text
MySQL > select bitmap_to_string(bitmap_empty());
+----------------------------------+
| bitmap_to_string(bitmap_empty()) |
+----------------------------------+
|                                  |
+----------------------------------+
```

示例 3：将包含一位的位图转换为字符串。

```Plain Text
MySQL > select bitmap_to_string(to_bitmap(1));
+--------------------------------+
| bitmap_to_string(to_bitmap(1)) |
+--------------------------------+
| 1                              |
+--------------------------------+
```

示例 4：将包含两位的位图转换为字符串。

```Plain Text
MySQL > select bitmap_to_string(bitmap_or(to_bitmap(1), to_bitmap(2)));
+---------------------------------------------------------+
| bitmap_to_string(bitmap_or(to_bitmap(1), to_bitmap(2))) |
+---------------------------------------------------------+
| 1,2                                                     |
+---------------------------------------------------------+
```
