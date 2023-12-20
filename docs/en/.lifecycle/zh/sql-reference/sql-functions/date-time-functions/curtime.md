---
displayed_sidebar: English
---

# curtime,current_time

## 描述

获取当前时间并返回TIME类型的值。

此函数在不同的时区可能会返回不同的结果。有关详细信息，请参阅[配置时区](../../../administration/timezone.md)。

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

## 关键词

CURTIME,CURRENT_TIME