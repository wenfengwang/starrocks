---
displayed_sidebar: "Chinese"
---

# now, current_timestamp, localtime, localtimestamp

## 描述

返回当前日期和时间。从v3.1开始，结果精确到微秒。

此函数对不同的时区可能返回不同的结果。更多信息，请参阅[配置时区](../../../administration/timezone.md)。

## 语法

```Haskell
DATETIME NOW()
```

## 例子

```Plain Text
MySQL > select now();
+---------------------+
| now()               |
+---------------------+
| 2019-05-27 15:58:25 |
+---------------------+

-- 自v3.1版本以来，结果精确到微秒。
MySQL > select now();
+----------------------------+
| now()                      |
+----------------------------+
| 2023-11-18 12:54:34.878000 |
+----------------------------+
```

## 关键词

NOW, now