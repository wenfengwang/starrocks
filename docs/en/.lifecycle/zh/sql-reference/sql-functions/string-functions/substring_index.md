---
displayed_sidebar: English
---

# substring_index

## 描述

提取出现分隔符`count`次数之前或之后的子字符串。

- 如果 `count` 为正数，则从字符串开头开始计数，该函数返回第`count`个分隔符之前的子字符串。例如，`select substring_index('https://www.starrocks.io', '.', 2);` 返回第二个 `.` 分隔符之前的子字符串，即 `https://www.starrocks`。

- 如果 `count` 为负数，则从字符串末尾开始计数，该函数返回第`count`个分隔符之后的子字符串。例如，`select substring_index('https://www.starrocks.io', '.', -2);` 返回第二个 `.` 分隔符之后的子字符串，即 `starrocks.io`。

如果任何输入参数为 null，则返回 NULL。

此函数从 v3.2 版本开始支持。

## 语法

```Haskell
VARCHAR substring_index(VARCHAR str, VARCHAR delimiter, INT count)
```

## 参数

- `str`：必填，要拆分的字符串。
- `delimiter`：必填，用于拆分字符串的分隔符。
- `count`：必填，分隔符的位置。该值不能为 0。否则将返回 NULL。如果该值大于字符串中实际的分隔符数量，则返回整个字符串。

## 返回值

返回 VARCHAR 值。

## 例子

```Plain Text
-- 返回第二个 "." 分隔符之前的子字符串。
mysql> select substring_index('https://www.starrocks.io', '.', 2);
+-----------------------------------------------------+
| substring_index('https://www.starrocks.io', '.', 2) |
+-----------------------------------------------------+
| https://www.starrocks                               |
+-----------------------------------------------------+

-- count 为负数。
mysql> select substring_index('https://www.starrocks.io', '.', -2);
+------------------------------------------------------+
| substring_index('https://www.starrocks.io', '.', -2) |
+------------------------------------------------------+
| starrocks.io                                         |
+------------------------------------------------------+

mysql> select substring_index("hello world", " ", 1);
+----------------------------------------+
| substring_index("hello world", " ", 1) |
+----------------------------------------+
| hello                                  |
+----------------------------------------+

mysql> select substring_index("hello world", " ", -1);
+-----------------------------------------+
| substring_index('hello world', ' ', -1) |
+-----------------------------------------+
| world                                   |
+-----------------------------------------+

-- count 为 0，返回 NULL。
mysql> select substring_index("hello world", " ", 0);
+----------------------------------------+
| substring_index('hello world', ' ', 0) |
+----------------------------------------+
| NULL                                   |
+----------------------------------------+

-- count 大于字符串中空格的数量，返回整个字符串。
mysql> select substring_index("hello world", " ", 2);
+----------------------------------------+
| substring_index("hello world", " ", 2) |
+----------------------------------------+
| hello world                            |
+----------------------------------------+

-- count 大于字符串中空格的数量，返回整个字符串。
mysql> select substring_index("hello world", " ", -2);
+-----------------------------------------+
| substring_index("hello world", " ", -2) |
+-----------------------------------------+
| hello world                             |
+-----------------------------------------+
```

## 关键词

substring_index
