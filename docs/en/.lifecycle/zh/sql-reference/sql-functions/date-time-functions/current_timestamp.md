---
displayed_sidebar: English
---

# current_timestamp

## 描述

获取当前日期，如果是 DATETIME 类型则返回一个值。

自 v3.1 起，结果精确到微秒。

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

-- 自 v3.1 起，结果精确到微秒。
MySQL > select current_timestamp();
+----------------------------+
| current_timestamp()        |
+----------------------------+
| 2023-11-18 12:58:05.375000 |
+----------------------------+
```

## 关键字

CURRENT_TIMESTAMP, CURRENT, TIMESTAMP