---
displayed_sidebar: English
---

# to_date

## 描述

将 DATETIME 值转换为日期。

## 语法

```Haskell
DATE TO_DATE(DATETIME datetime)
```

## 示例

```Plain
MySQL > select to_date("2020-02-02 00:00:00");
+--------------------------------+
| to_date('2020-02-02 00:00:00') |
+--------------------------------+
| 2020-02-02                     |
+--------------------------------+
```

## 关键字

TO_DATE