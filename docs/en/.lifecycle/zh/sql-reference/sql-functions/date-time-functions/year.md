---
displayed_sidebar: English
---

# YEAR

## 描述

返回日期中的年份部分，取值范围为 1000 到 9999。

`date` 参数必须是 DATE 或 DATETIME 类型。

## 语法

```Haskell
INT YEAR(DATETIME date)
```

## 示例

```Plain
MySQL > select year('1987-01-01');
+-----------------------------+
| year('1987-01-01 00:00:00') |
+-----------------------------+
|                        1987 |
+-----------------------------+
```

## 关键字

YEAR