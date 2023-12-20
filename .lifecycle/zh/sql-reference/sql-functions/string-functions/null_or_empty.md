---
displayed_sidebar: English
---

# 字符串为空或NULL时

## 描述

当字符串为空或为NULL时，此函数返回真（true）。其他情况下，返回假（false）。

## 语法

```Haskell
BOOLEAN NULL_OR_EMPTY (VARCHAR str)
```

## 示例

```Plain
MySQL > select null_or_empty(null);
+---------------------+
| null_or_empty(NULL) |
+---------------------+
|                   1 |
+---------------------+

MySQL > select null_or_empty("");
+-------------------+
| null_or_empty('') |
+-------------------+
|                 1 |
+-------------------+

MySQL > select null_or_empty("a");
+--------------------+
| null_or_empty('a') |
+--------------------+
|                  0 |
+--------------------+
```

## 关键字

NULL_OR_EMPTY
