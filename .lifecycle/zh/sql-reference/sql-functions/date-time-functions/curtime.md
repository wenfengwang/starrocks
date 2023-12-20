---
displayed_sidebar: English
---

# 获取当前时间，CURRENT_TIME

## 描述

此函数用于获取当前时间，并返回一个TIME类型的值。

该函数可能会根据不同的时区返回不同的结果。更多信息请参见[配置时区](../../../administration/timezone.md)。

## 语法

```Haskell
TIME CURTIME()
```

## 示例

```Plain
MySQL > select current_time();
+----------------+
| current_time() |
+----------------+
| 15:25:47       |
+----------------+
```

## 关键字

CURTIME，CURRENT_TIME
