---
displayed_sidebar: English
---

# sleep

## 描述

延迟操作的执行特定时间段（以秒为单位），并返回一个 BOOLEAN 值以指示是否在无中断的情况下完成了延迟。如果延迟完成且无中断，则返回 `1`。否则，返回 `0`。

## 语法

```Haskell
BOOLEAN sleep(INT x);
```

## 参数

`x`：你想要延迟操作执行的时间长度。它必须是 INT 类型。单位：秒。如果输入是 NULL，则立即返回 NULL，不进行延迟。

## 返回值

返回 BOOLEAN 类型的值。

## 示例

```Plain
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

## 关键字

SLEEP, sleep