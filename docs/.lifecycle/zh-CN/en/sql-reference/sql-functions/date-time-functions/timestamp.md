---
displayed_sidebar: "Chinese"
---

# 时间戳

## 描述

返回日期或日期时间表达式的 DATETIME 值。

## 语法

```Haskell
DATETIME timestamp(DATETIME|DATE expr);
```

## 参数

`expr`: 要转换的时间表达式。它必须是 DATETIME 或 DATE 类型。

## 返回值

返回 DATETIME 值。如果输入时间为空或不存在，比如 `2021-02-29`，则返回 NULL。

## 示例

```Plain Text
select timestamp("2019-05-27");
+-------------------------+
| timestamp('2019-05-27') |
+-------------------------+
| 2019-05-27 00:00:00     |
+-------------------------+
1 row in set (0.00 sec)
```