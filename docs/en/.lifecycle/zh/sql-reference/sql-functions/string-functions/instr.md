---
displayed_sidebar: English
---

# instr

## 描述

此函数返回字符串 str 在子字符串 substr 中首次出现的位置（从 1 开始计数，单位为字符）。如果在 substr 中未找到 str，则此函数将返回 0。

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

## 关键字

INSTR