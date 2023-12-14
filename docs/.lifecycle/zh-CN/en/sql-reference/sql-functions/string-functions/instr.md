---
displayed_sidebar: "English"
---

# instr

## 描述

此函数返回str第一次出现在substr中的位置（从1开始计数，以字符为单位）。如果substr中找不到str，则此函数将返回0。

## 语法

```Haskell
INT instr(VARCHAR str, VARCHAR substr)
```

## 示例

```Plain Text
MySQL > select instr("abc", "b");
+-------------------+
| instr('abc', 'b') |
+-------------------+
|                 2 |
+-------------------+

MySQL > select instr("abc", "d");
+-------------------+
| instr('abc', 'd') |
+-------------------+
|                 0 |
+-------------------+
```

## 关键词

INSTR