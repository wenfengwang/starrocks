---
displayed_sidebar: English
---

# 子字符串索引

## 描述

此函数用于提取分隔符出现指定次数之前或之后的子字符串。

- 如果 count 为正数，计数则从字符串的开头开始，该函数将返回第 count 个分隔符之前的子字符串。例如，执行 substring_index('https://www.starrocks.io', '.', 2); 将返回第二个“.”分隔符之前的子字符串，即 https://www.starrocks。

- 如果 count 为负数，计数则从字符串的末尾开始，该函数将返回第 count 个分隔符之后的子字符串。例如，执行 substring_index('https://www.starrocks.io', '.', -2); 将返回倒数第二个“.”分隔符之后的子字符串，即 starrocks.io。

如果任何输入参数为 null，则返回 NULL。

该函数从 v3.2 版本开始支持。

## 语法

```Haskell
VARCHAR substring_index(VARCHAR str, VARCHAR delimiter, INT count)
```

## 参数

- str：必须，你想要拆分的字符串。
- delimiter：必须，用于拆分字符串的分隔符。
- count：必须，分隔符的位置。该值不能为 0，否则返回 NULL。如果该值大于字符串中分隔符的实际数量，则返回整个字符串。

## 返回值

返回一个 VARCHAR 类型的值。

## 示例

```Plain
-- Return the substring that precedes the second "." delimiter.
mysql> select substring_index('https://www.starrocks.io', '.', 2);
+-----------------------------------------------------+
| substring_index('https://www.starrocks.io', '.', 2) |
+-----------------------------------------------------+
| https://www.starrocks                               |
+-----------------------------------------------------+

-- The count is negative.
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

-- Count is 0 and NULL is returned.
mysql> select substring_index("hello world", " ", 0);
+----------------------------------------+
| substring_index('hello world', ' ', 0) |
+----------------------------------------+
| NULL                                   |
+----------------------------------------+

-- Count is greater than the number of spaces in the string and the entire string is returned.
mysql> select substring_index("hello world", " ", 2);
+----------------------------------------+
| substring_index("hello world", " ", 2) |
+----------------------------------------+
| hello world                            |
+----------------------------------------+

-- Count is greater than the number of spaces in the string and the entire string is returned.
mysql> select substring_index("hello world", " ", -2);
+-----------------------------------------+
| substring_index("hello world", " ", -2) |
+-----------------------------------------+
| hello world                             |
+-----------------------------------------+
```

## 关键词

子字符串索引
