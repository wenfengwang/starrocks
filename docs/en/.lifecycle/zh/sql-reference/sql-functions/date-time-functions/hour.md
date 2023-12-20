---
displayed_sidebar: English
---

# HOUR

## 描述

返回给定日期的小时数。返回值范围从0到23。

`date` 参数必须是 DATE 或 DATETIME 类型。

## 语法

```Haskell
INT HOUR(DATETIME|DATE date)
```

## 示例

```Plain
MySQL > select hour('2018-12-31 23:59:59');
+-----------------------------+
| hour('2018-12-31 23:59:59') |
+-----------------------------+
|                          23 |
+-----------------------------+
```

## 关键字

HOUR