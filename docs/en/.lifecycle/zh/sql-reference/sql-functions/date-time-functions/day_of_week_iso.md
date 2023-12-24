---
displayed_sidebar: English
---

# dayofweek_iso

## 描述

返回指定日期的 ISO 标准星期几，以整数形式表示，范围为 `1` 到 `7`。在该标准中，`1` 表示星期一，`7` 表示星期日。

## 语法

```Haskell
INT DAY_OF_WEEK_ISO(DATETIME date)
```

## 参数

`date`：要转换的日期。必须是 DATE 或 DATETIME 类型。

## 例子

以下示例返回日期 `2023-01-01` 的 ISO 标准星期几：

```SQL
MySQL > select dayofweek_iso('2023-01-01');
+-----------------------------+
| dayofweek_iso('2023-01-01') |
+-----------------------------+
|                           7 |
+-----------------------------+
```

## 关键字

DAY_OF_WEEK_ISO
