---
displayed_sidebar: English
---

# 小时

## 描述

返回给定日期的小时。返回值的范围是从 0 到 23。

`date` 参数必须是 DATE 或 DATETIME 类型。

## 语法

```Haskell
INT HOUR(DATETIME|DATE date)
```

## 例子

```Plain Text
MySQL > select hour('2018-12-31 23:59:59');
+-----------------------------+
| hour('2018-12-31 23:59:59') |
+-----------------------------+
|                          23 |
+-----------------------------+
```

## 关键词

HOUR