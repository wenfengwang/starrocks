---
displayed_sidebar: "Chinese"
---

# utc_timestamp

## 描述

返回当前UTC日期和时间作为'YYYY-MM-DD HH:MM:SS'或'YYYYMMDDHHMMSS'格式的值，取决于函数的使用方式，例如在字符串或数字上下文中。

## 语法

```Haskell
DATETIME UTC_TIMESTAMP()
```

## 示例

```Plain Text
MySQL > select utc_timestamp(), utc_timestamp() + 1;
+---------------------+---------------------+
| utc_timestamp()     | utc_timestamp() + 1 |
+---------------------+---------------------+
| 2019-07-10 12:31:18 |      20190710123119 |
+---------------------+---------------------+

-- 从v3.1开始结果精确到微秒。
select utc_timestamp();
+----------------------------+
| utc_timestamp()            |
+----------------------------+
| 2023-11-18 04:59:14.561000 |
+----------------------------+
```

`utc_timestamp() + N` 表示在当前时间上加上 `N` 秒。

## 关键词

UTC_TIMESTAMP,UTC,TIMESTAMP