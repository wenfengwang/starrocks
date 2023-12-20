---
displayed_sidebar: English
---

# 分钟

## 描述

返回指定日期的分钟数。返回值的范围为0至59。

日期参数必须为 DATE 或 DATETIME 类型。

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
