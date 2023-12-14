---
displayed_sidebar: "Chinese"
---

# null_or_empty

## 描述

当字符串为空或为NULL时，此函数返回true。 否则，它返回false。

## 语法

```Haskell
BOOLEAN NULL_OR_EMPTY (VARCHAR str)
```

## 示例

```Plain Text
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

## 关键词

NULL_OR_EMPTY