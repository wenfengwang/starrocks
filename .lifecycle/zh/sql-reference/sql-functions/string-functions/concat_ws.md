---
displayed_sidebar: English
---

# concat_ws

## 描述

此函数将第一个参数 sep 作为分隔符，用以将第二个参数及之后的参数拼接成一个字符串。如果分隔符是 NULL，那么结果也是 NULL。concat_ws 不会忽略空字符串，但它会忽略 NULL 值。

## 语法

```Haskell
VARCHAR concat_ws(VARCHAR sep, VARCHAR str,...)
```

## 示例

```Plain
MySQL > select concat_ws("Rock", "Star", "s");
+--------------------------------+
| concat_ws('Rock', 'Star', 's') |
+--------------------------------+
| StarRocks                      |
+--------------------------------+

MySQL > select concat_ws(NULL, "Star", "s");
+------------------------------+
| concat_ws(NULL, 'Star', 's') |
+------------------------------+
| NULL                         |
+------------------------------+
1 row in set (0.01 sec)

MySQL > StarRocks > select concat_ws("Rock", "Star", NULL, "s");
+--------------------------------------+
| concat_ws('Rock', 'Star', NULL, 's') |
+--------------------------------------+
| StarRocks                            |
+--------------------------------------+
1 row in set (0.04 sec)
```

## 关键词

CONCAT_WS, CONCAT, WS
