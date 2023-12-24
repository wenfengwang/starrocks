---
displayed_sidebar: English
---

# instr

## 描述

此函数返回 str 在 substr 中首次出现的位置（从第1个字符开始计数）。如果在 substr 中找不到 str，则该函数将返回 0。

## 语法

```Haskell
INT instr(VARCHAR str, VARCHAR substr)
```

## 例子

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

INSTR （英语）
