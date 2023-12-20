---
displayed_sidebar: English
---

# MINUTE

## 描述

返回给定日期的分钟数。返回值范围从0到59。

`date` 参数必须是 DATE 或 DATETIME 类型。

## 语法

```Haskell
INT MINUTE(DATETIME|DATE date)
```

## 示例

```Plain
MySQL > select minute('2018-12-31 23:59:59');
+-----------------------------+
|minute('2018-12-31 23:59:59')|
+-----------------------------+
|                          59 |
+-----------------------------+
```

## 关键字

MINUTE