---
displayed_sidebar: English
---

# ends_with

## 描述

如果字符串以指定的后缀结尾，则返回 `true`。否则，返回 `false`。如果参数为 NULL，则结果为 NULL。

## 语法

```Haskell
BOOLEAN ENDS_WITH (VARCHAR str, VARCHAR suffix)
```

## 示例

```Plain
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

## 关键字

ENDS_WITH