---
displayed_sidebar: English
---

# 以...结尾

## 描述

如果一个字符串以特定的后缀结束，则返回真（true）。否则，返回假（false）。如果参数是空值（NULL），则结果也是空值（NULL）。

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
