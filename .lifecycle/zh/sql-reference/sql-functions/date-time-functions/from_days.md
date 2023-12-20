---
displayed_sidebar: English
---

# 从天数计算日期

## 描述

从公元前 0000 年 01 月 01 日起算返回对应的日期。

## 语法

```Haskell
DATE FROM_DAYS(INT N)
```

## 示例

```Plain
MySQL > select from_days(730669);
+-------------------+
| from_days(730669) |
+-------------------+
| 2000-07-03        |
+-------------------+
```

## 关键字

FROM_DAYS，FROM，DAYS
