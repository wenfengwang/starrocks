---
displayed_sidebar: "Chinese"
---

# curtime,current_time

## 描述

获取当前时间并返回一个TIME类型的值。

此函数在不同的时区可能返回不同的结果。更多信息请参阅[配置时区](../../../administration/timezone.md)。

## 语法

```Haskell
TIME CURTIME()
```

## 示例

```Plain Text
MySQL > select current_time();
+----------------+
| current_time() |
+----------------+
| 15:25:47       |
+----------------+
```

## 关键字

CURTIME,CURRENT_TIME