---
displayed_sidebar: English
---

# weeks_sub

## 描述

返回减去指定周数后的日期值。

## 语法

```Haskell
DATETIME weeks_sub(DATETIME expr1, INT expr2);
```

## 参数

- `expr1`：原始日期。必须为 `DATETIME` 类型。

- `expr2`：周数。必须为 `INT` 类型。

## 返回值

返回 `DATETIME` 类型的值。 

如果日期不存在，则返回 `NULL`。

## 例子

```Plain
select weeks_sub('2022-12-22',2);
+----------------------------+
| weeks_sub('2022-12-22', 2) |
+----------------------------+
|        2022-12-08 00:00:00 |
+----------------------------+