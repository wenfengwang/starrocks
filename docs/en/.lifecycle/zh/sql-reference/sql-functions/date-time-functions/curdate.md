---
displayed_sidebar: English
---

# curdate,current_date

## 描述

获取当前日期并返回DATE类型的值。

## 语法

```Haskell
DATE CURDATE()
```

## 示例

```Plain
MySQL > SELECT CURDATE();
+------------+
| curdate()  |
+------------+
| 2022-12-20 |
+------------+

SELECT CURRENT_DATE();
+----------------+
| current_date() |
+----------------+
| 2022-12-20     |
+----------------+


MySQL > SELECT CURDATE() + 0;
+---------------+
| CURDATE() + 0 |
+---------------+
|      20221220 |
+---------------+
```