---
displayed_sidebar: "Chinese"
---

# 分钟

## 描述

返回给定日期的分钟数。返回值范围为0到59。

`date`参数必须是DATE或DATETIME类型。

## 语法

```Haskell
INT MINUTE(DATETIME|DATE date)
```

## 示例

```Plain Text
MySQL > select minute('2018-12-31 23:59:59');
+-----------------------------+
|minute('2018-12-31 23:59:59')|
+-----------------------------+
|                          59 |
+-----------------------------+
```

## 关键词

分钟