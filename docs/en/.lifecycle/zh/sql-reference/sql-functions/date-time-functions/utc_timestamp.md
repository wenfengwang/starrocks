---
displayed_sidebar: English
---

# utc_timestamp

## 描述

返回当前 UTC 日期和时间，格式为 'YYYY-MM-DD HH:MM:SS' 或 'YYYYMMDDHHMMSS'，具体格式取决于函数的使用场景，例如在字符串或数字上下文中。

## 语法

```Haskell
DATETIME UTC_TIMESTAMP()
```

## 示例

```Plain
MySQL > select utc_timestamp(),utc_timestamp() + 1;
+---------------------+---------------------+
| utc_timestamp()     | utc_timestamp() + 1 |
+---------------------+---------------------+
| 2019-07-10 12:31:18 |      20190710123119 |
+---------------------+---------------------+

-- 自 v3.1 起，结果精确到微秒。
select utc_timestamp();
+----------------------------+
| utc_timestamp()            |
+----------------------------+
| 2023-11-18 04:59:14.561000 |
+----------------------------+
```

`utc_timestamp() + N` 表示在当前时间基础上加上 `N` 秒。

## 关键字

UTC_TIMESTAMP,UTC,TIMESTAMP