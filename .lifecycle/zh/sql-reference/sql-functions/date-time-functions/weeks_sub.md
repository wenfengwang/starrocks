---
displayed_sidebar: English
---

# 周数减法

## 描述

返回从指定日期减去一定周数后的值。

## 语法

```Haskell
DATETIME weeks_sub(DATETIME expr1, INT expr2);
```

## 参数

- expr1：原始日期。必须是 DATETIME 类型。

- expr2：减去的周数。必须是 INT 类型。

## 返回值

返回 DATETIME 类型。

如果日期不存在，则返回 NULL。

## 示例

```Plain
select weeks_sub('2022-12-22',2);
+----------------------------+
| weeks_sub('2022-12-22', 2) |
+----------------------------+
|        2022-12-08 00:00:00 |
+----------------------------+
```
