---
displayed_sidebar: English
---

# from_days

## 描述

返回从 0000-01-01 开始经过指定天数后的日期。

## 语法

```Haskell
DATE FROM_DAYS(INT N)
```

## 例子

```Plain Text
MySQL > select from_days(730669);
+-------------------+
| from_days(730669) |
+-------------------+
| 2000-07-03        |
+-------------------+
```

## 关键词

FROM_DAYS, FROM, DAYS