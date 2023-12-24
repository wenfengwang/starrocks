---
displayed_sidebar: English
---

# 分钟

## 描述

返回给定日期的分钟数。返回值范围从 0 到 59。

`date` 参数必须是 DATE 或 DATETIME 类型。

## 语法

```Haskell
INT MINUTE(DATETIME|DATE date)
```

## 例子

```Plain Text
MySQL > select minute('2018-12-31 23:59:59');
+-----------------------------+
|minute('2018-12-31 23:59:59')|
+-----------------------------+
|                          59 |
+-----------------------------+
```

## 关键词

MINUTE