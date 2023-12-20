---
displayed_sidebar: English
---

# 当前时间戳

## 描述

获取当前日期并以 DATETIME 类型返回值。

自 v3.1 起，结果的精确度达到微秒级。

## 语法

```Haskell
DATETIME CURRENT_TIMESTAMP()
```

## 示例

```Plain
MySQL > select current_timestamp();
+---------------------+
| current_timestamp() |
+---------------------+
| 2019-05-27 15:59:33 |
+---------------------+

-- The result is accurate to the microsecond since v3.1.
MySQL > select current_timestamp();
+----------------------------+
| current_timestamp()        |
+----------------------------+
| 2023-11-18 12:58:05.375000 |
+----------------------------+
```

## 关键字

CURRENT_TIMESTAMP、CURRENT、TIMESTAMP
