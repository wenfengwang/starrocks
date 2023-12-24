---
displayed_sidebar: English
---

# concat_ws

## 描述

此函数使用第一个参数 sep 作为分隔符，将第二个参数与其他参数组合成一个字符串。如果分隔符为 NULL，则结果也为 NULL。concat_ws 不会跳过空字符串，但会跳过 NULL 值。

## 语法

```Haskell
VARCHAR concat_ws(VARCHAR sep, VARCHAR str,...)
```

## 例子

```Plain Text
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
1 行受影响 (0.01 秒)

MySQL > select concat_ws("Rock", "Star", NULL, "s");
+--------------------------------------+
| concat_ws('Rock', 'Star', NULL, 's') |
+--------------------------------------+
| StarRocks                            |
+--------------------------------------+
1 行受影响 (0.04 秒)
```

## 关键词

CONCAT_WS，CONCAT，WS