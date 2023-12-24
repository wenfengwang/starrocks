---
displayed_sidebar: English
---

# CURDATE, CURRENT_DATE

## 描述

获取当前日期并返回 DATE 类型的值。

## 语法

```Haskell
DATE CURDATE()
```

## 例子

```Plain Text
MySQL > SELECT CURDATE();
+------------+
| CURDATE()  |
+------------+
| 2022-12-20 |
+------------+

SELECT CURRENT_DATE();
+----------------+
| CURRENT_DATE() |
+----------------+
| 2022-12-20     |
+----------------+


MySQL > SELECT CURDATE() + 0;
+---------------+
| CURDATE() + 0 |
+---------------+
|      20221220 |
+---------------+