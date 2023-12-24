---
displayed_sidebar: English
---

# ends_with

## 描述

如果一个字符串以指定的后缀结尾，则返回 `true`。否则返回 `false`。如果参数为 NULL，则结果也为 NULL。

## 语法

```Haskell
BOOLEAN ENDS_WITH (VARCHAR str, VARCHAR suffix)
```

## 例子

```Plain Text
MySQL > select ends_with("Hello starrocks", "starrocks");
+-----------------------------------+
| ends_with('Hello starrocks', 'starrocks') |
+-----------------------------------+
|                                 1 |
+-----------------------------------+

MySQL > select ends_with("Hello starrocks", "Hello");
+-----------------------------------+
| ends_with('Hello starrocks', 'Hello') |
+-----------------------------------+
|                                 0 |
+-----------------------------------+
```

## 关键词

ENDS_WITH
