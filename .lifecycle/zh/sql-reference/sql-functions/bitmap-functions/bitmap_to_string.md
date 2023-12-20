---
displayed_sidebar: English
---

# 位图转字符串

## 描述

将输入的位图转换成由逗号 (,) 分隔的字符串。此字符串包含位图中的所有比特位。如果输入为 null，则返回 null。

## 语法

```Haskell
VARCHAR BITMAP_TO_STRING(BITMAP input)
```

## 参数

输入：欲转换的位图。

## 返回值

返回一个 VARCHAR 类型的值。

## 示例

示例 1：输入为 null，因此返回 null。

```Plain
MySQL > select bitmap_to_string(null);
+------------------------+
| bitmap_to_string(NULL) |
+------------------------+
| NULL                   |
+------------------------+
```

示例 2：输入为空，因此返回一个空字符串。

```Plain
MySQL > select bitmap_to_string(bitmap_empty());
+----------------------------------+
| bitmap_to_string(bitmap_empty()) |
+----------------------------------+
|                                  |
+----------------------------------+
```

示例 3：将只包含一个比特位的位图转换为字符串。

```Plain
MySQL > select bitmap_to_string(to_bitmap(1));
+--------------------------------+
| bitmap_to_string(to_bitmap(1)) |
+--------------------------------+
| 1                              |
+--------------------------------+
```

示例 4：将包含两个比特位的位图转换为字符串。

```Plain
MySQL > select bitmap_to_string(bitmap_or(to_bitmap(1), to_bitmap(2)));
+---------------------------------------------------------+
| bitmap_to_string(bitmap_or(to_bitmap(1), to_bitmap(2))) |
+---------------------------------------------------------+
| 1,2                                                     |
+---------------------------------------------------------+
```
