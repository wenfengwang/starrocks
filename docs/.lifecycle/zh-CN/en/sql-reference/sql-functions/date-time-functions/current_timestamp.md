---
显示的侧边栏："英语"
---

# current_timestamp

## 描述

获取当前日期并以DATETIME类型返回值。

自v3.1以来，结果精确到微秒。

## 语法

```Haskell
DATETIME CURRENT_TIMESTAMP()
```

## 示例

```Plain Text
MySQL > select current_timestamp();
+---------------------+
| current_timestamp() |
+---------------------+
| 2019-05-27 15:59:33 |
+---------------------+

-- 自v3.1以来，结果精确到微秒。
MySQL > select current_timestamp();
+----------------------------+
| current_timestamp()        |
+----------------------------+
| 2023-11-18 12:58:05.375000 |
+----------------------------+
```

## 关键词

CURRENT_TIMESTAMP,CURRENT,TIMESTAMP