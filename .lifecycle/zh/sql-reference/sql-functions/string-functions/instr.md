---
displayed_sidebar: English
---

# 指令

## 描述

此函数用于返回字符串 str 在 substr 中首次出现的位置（位置计数从 1 开始，以字符为计量单位）。如果 substr 中未找到 str，则函数将返回 0。

## 语法

```Haskell
INT instr(VARCHAR str, VARCHAR substr)
```

## 示例

```Plain
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
