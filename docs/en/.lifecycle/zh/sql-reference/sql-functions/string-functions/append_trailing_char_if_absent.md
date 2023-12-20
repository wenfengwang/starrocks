---
displayed_sidebar: English
---

# append_trailing_char_if_absent

## 描述

如果字符串 str 不为空且末尾没有 trailing_char 字符，则会在其末尾追加 trailing_char 字符。trailing_char 只能包含一个字符。如果包含多个字符，该函数将返回 NULL。

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