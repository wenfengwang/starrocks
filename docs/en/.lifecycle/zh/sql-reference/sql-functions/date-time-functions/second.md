---
displayed_sidebar: English
---

# 秒

## 描述

返回给定日期的秒部分。返回值范围从 0 到 59。

`date` 参数必须是 DATE 或 DATETIME 类型。

## 语法

```Haskell
INT SECOND(DATETIME date)
```

## 例子

```Plain Text
MySQL > select second('2018-12-31 23:59:59');
+-----------------------------+
|second('2018-12-31 23:59:59')|
+-----------------------------+
|                          59 |
+-----------------------------+
```

## 关键词

秒
