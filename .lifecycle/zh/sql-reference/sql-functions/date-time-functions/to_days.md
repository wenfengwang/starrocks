---
displayed_sidebar: English
---

# 今天

## 描述

返回指定日期与公元前0000年1月1日之间的天数差。

日期参数必须是 DATE 或 DATETIME 类型。

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

## 关键词

TO_DAYS、TO、DAYS
