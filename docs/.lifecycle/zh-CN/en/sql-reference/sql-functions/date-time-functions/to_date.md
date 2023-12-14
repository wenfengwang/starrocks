---
displayed_sidebar: "Chinese"
---

# to_date

## 描述

将DATETIME值转换为日期。

## 语法

```Haskell
DATE TO_DATE(DATETIME datetime)
```

## 例子

```Plain Text
MySQL > select to_date("2020-02-02 00:00:00");
+--------------------------------+
| to_date('2020-02-02 00:00:00') |
+--------------------------------+
| 2020-02-02                     |
+--------------------------------+
```

## 关键词

TO_DATE