---
displayed_sidebar: English
---

# to_days

## 描述

返回一个日期与 0000-01-01 之间的天数。

`date` 参数必须是 DATE 或 DATETIME 类型。

## 语法

```Haskell
INT TO_DAYS(DATETIME date)
```

## 例子

```Plain Text
MySQL > select to_days('2007-10-07');
+-----------------------+
| to_days('2007-10-07') |
+-----------------------+
|                733321 |
+-----------------------+
```

## 关键词

TO_DAYS, TO, DAYS