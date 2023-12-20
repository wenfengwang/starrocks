---
displayed_sidebar: English
---

# 暂停执行

## 描述

暂停执行某个操作指定的时间（单位为秒），并返回一个布尔值（BOOLEAN），用以表明暂停是否在没有中断的情况下完成。如果暂停完成且未被中断，返回 1。否则，返回 0。

## 语法

```Haskell
BOOLEAN sleep(INT x);
```

## 参数

x：你想要延迟操作执行的时间长度。该参数必须为整型（INT）数据。单位为秒。如果输入值为 NULL，则会立即返回 NULL，不进行暂停。

## 返回值

返回布尔类型（BOOLEAN）的值。

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

## 关键词

SLEEP, sleep
