---
displayed_sidebar: English
---

# char

## 描述

CHAR() 函数根据 ASCII 表返回给定整数值的字符表示。

## 语法

```Haskell
char(n)
```

## 参数

- `n`：整数值

## 返回值

返回一个 VARCHAR 类型的值。

## 示例

```Plain
> select char(77);
+----------+
| char(77) |
+----------+
| M        |
+----------+
```

## 关键字

CHAR