---
displayed_sidebar: English
---

# to_days

## 描述

返回一个日期与公元 0000-01-01 之间的天数差。

`date` 参数必须是 DATE 或 DATETIME 类型。

## 语法

```Haskell
INT TO_DAYS(DATETIME date)
```

## 示例

```Plain
MySQL > select to_days('2007-10-07');
+-----------------------+
| to_days('2007-10-07') |
+-----------------------+
|                733321 |
+-----------------------+
```

## 关键字

TO_DAYS, TO, DAYS