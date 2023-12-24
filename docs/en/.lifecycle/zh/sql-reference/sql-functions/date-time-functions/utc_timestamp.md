---
displayed_sidebar: English
---

# utc_timestamp

## 描述

返回当前的UTC日期和时间数值，格式为'YYYY-MM-DD HH:MM:SS'或'YYYYMMDDHHMMSS'，取决于函数的用法，例如在字符串或数字上下文中。

## 语法

```Haskell
DATETIME UTC_TIMESTAMP()
```

## 例子

```Plain Text
MySQL > select utc_timestamp(),utc_timestamp() + 1;
+---------------------+---------------------+
| utc_timestamp()     | utc_timestamp() + 1 |
+---------------------+---------------------+
| 2019-07-10 12:31:18 |      20190710123119 |
+---------------------+---------------------+

-- 自3.1版本以来，结果精确到微秒。
select utc_timestamp();
+----------------------------+
| utc_timestamp()            |
+----------------------------+
| 2023-11-18 04:59:14.561000 |
+----------------------------+
```

`utc_timestamp() + N` 表示将`N`秒添加到当前时间。

## 关键词

UTC_TIMESTAMP，UTC，时间戳