---
displayed_sidebar: "Chinese"
---

# append_trailing_char_if_absent

## 描述

如果str字符串不为空且不以trailing_char字符结尾，则将trailing_char字符添加到末尾。trailing_char只能包含一个字符。如果包含多个字符，此函数将返回NULL。

## 语法

```Haskell
VARCHAR append_trailing_char_if_absent(VARCHAR str, VARCHAR trailing_char)
```

## 示例

```Plain Text
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

## 关键词

APPEND_TRAILING_CHAR_IF_ABSENT