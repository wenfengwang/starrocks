---
displayed_sidebar: English
---

# 至今

## 描述

将 DATETIME 值转换成日期格式。

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
