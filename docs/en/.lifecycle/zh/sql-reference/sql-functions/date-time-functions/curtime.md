---
displayed_sidebar: English
---

# curtime, current_time

## 描述

获取当前时间并返回 TIME 类型的值。

该函数可能会针对不同的时区返回不同的结果。更多信息，请参阅 [配置时区](../../../administration/timezone.md)。

## 语法

```Haskell
TIME CURTIME()
```

## 例子

```Plain Text
MySQL > select current_time();
+----------------+
| current_time() |
+----------------+
| 15:25:47       |
+----------------+
```

## 关键词

CURTIME, CURRENT_TIME