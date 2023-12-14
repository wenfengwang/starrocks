---
displayed_sidebar: "中文"
---

# sleep

## 描述

延迟执行指定时间段（以秒为单位）的操作，并返回一个BOOLEAN值以指示睡眠是否未被中断。如果睡眠未被中断，则返回`1`。否则返回`0`。

## 语法

```Haskell
BOOLEAN sleep(INT x);
```

## 参数

`x`：您想要延迟操作执行的持续时间。它必须是INT类型。单位：秒。如果输入为NULL，则立即返回NULL而不进行睡眠。

## 返回值

返回一个BOOLEAN类型的值。

## 示例

```Plain Text
select sleep(3);
+----------+
| sleep(3) |
+----------+
|        1 |
+----------+
1 row in set (3.00 sec)

select sleep(NULL);
+-------------+
| sleep(NULL) |
+-------------+
|        NULL |
+-------------+
1 row in set (0.00 sec)
```

## 关键词

SLEEP, sleep