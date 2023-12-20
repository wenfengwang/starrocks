---
displayed_sidebar: English
---

# 如果字符串末尾缺少指定字符，则追加该字符

## 描述

如果字符串str非空且末尾没有trailing_char字符，则会在其末尾追加trailing_char字符。trailing_char只能是单个字符。如果包含多个字符，该函数将返回NULL。

## 语法

```Haskell
VARCHAR append_trailing_char_if_absent(VARCHAR str, VARCHAR trailing_char)
```

## 示例

```Plain
MySQL [test]> select append_trailing_char_if_absent('a','c');
+------------------------------------------+
|append_trailing_char_if_absent('a', 'c')  |
+------------------------------------------+
| ac                                       |
+------------------------------------------+
1 row in set (0.02 sec)

MySQL [test]> select append_trailing_char_if_absent('ac','c');
+-------------------------------------------+
|append_trailing_char_if_absent('ac', 'c')  |
+-------------------------------------------+
| ac                                        |
+-------------------------------------------+
1 row in set (0.00 sec)
```

## 关键字

APPEND_TRAILING_CHAR_IF_ABSENT
