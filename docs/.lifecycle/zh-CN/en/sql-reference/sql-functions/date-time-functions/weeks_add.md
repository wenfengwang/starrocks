```yaml
---
displayed_sidebar: "Chinese"
---

# weeks_add

## 描述

返回添加指定周数后的日期值。

## 语法

```Haskell
DATETIME weeks_add(DATETIME expr1, INT expr2);
```

## 参数

- `expr1`: 原始日期。必须是 `DATETIME` 类型。

- `expr2`: 周数。必须是 `INT` 类型。

## 返回值

返回 `DATETIME`。

如果日期不存在，则返回`NULL`。

## 示例

```Plain
select weeks_add('2022-12-20',2);
+----------------------------+
| weeks_add('2022-12-20', 2) |
+----------------------------+
|        2023-01-03 00:00:00 |
+----------------------------+
```