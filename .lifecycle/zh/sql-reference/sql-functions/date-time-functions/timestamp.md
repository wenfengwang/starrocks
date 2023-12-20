---
displayed_sidebar: English
---

# 时间戳

## 描述

返回一个日期或日期时间表达式的 DATETIME 值。

## 语法

```Haskell
DATETIME timestamp(DATETIME|DATE expr);
```

## 参数

expr：你想要转换的时间表达式。它必须是 DATETIME 或 DATE 类型。

## 返回值

返回一个 DATETIME 值。如果输入的时间为空或者不是一个有效日期，比如2021-02-29，那么将返回 NULL。

## 示例

```Plain
select timestamp("2019-05-27");
+-------------------------+
| timestamp('2019-05-27') |
+-------------------------+
| 2019-05-27 00:00:00     |
+-------------------------+
1 row in set (0.00 sec)
```
