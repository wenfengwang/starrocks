---
displayed_sidebar: English
---

# UTC时间戳

## 描述

根据函数的使用场景，返回当前UTC日期和时间的值，格式可能是'YYYY-MM-DD HH:MM:SS'或者'YYYYMMDDHHMMSS'，例如在字符串或数值上下文中。

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

-- The result is accurate to the microsecond since v3.1.
select utc_timestamp();
+----------------------------+
| utc_timestamp()            |
+----------------------------+
| 2023-11-18 04:59:14.561000 |
+----------------------------+
```

utc_timestamp() + N 表示在当前时间基础上增加N秒。

## 关键字

UTC_TIMESTAMP、UTC、时间戳
