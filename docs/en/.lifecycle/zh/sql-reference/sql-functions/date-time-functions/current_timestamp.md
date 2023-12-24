---
displayed_sidebar: English
---

# current_timestamp

## 描述

获取当前日期并返回一个 DATETIME 类型的值。

从 v3.1 版本开始，结果精确到微秒。

## 语法

```Haskell
DATETIME CURRENT_TIMESTAMP()
```

## 例子

```Plain Text
MySQL > select current_timestamp();
+---------------------+
| current_timestamp() |
+---------------------+
| 2019-05-27 15:59:33 |
+---------------------+

-- 自 v3.1 版本以来，结果精确到微秒。
MySQL > select current_timestamp();
+----------------------------+
| current_timestamp()        |
+----------------------------+
| 2023-11-18 12:58:05.375000 |
+----------------------------+
```

## 关键词

CURRENT_TIMESTAMP, CURRENT, TIMESTAMP
