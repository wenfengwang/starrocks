---
displayed_sidebar: "Chinese"
---

# dayofweek_iso

## 描述

返回指定日期的ISO标准星期几，表示为范围为 `1` 到 `7` 的整数。在这个标准中，`1` 表示星期一，`7` 表示星期日。

## 语法

```Haskell
INT DAY_OF_WEEK_ISO(DATETIME date)
```

## 参数

`date`: 要转换的日期。必须是日期或日期时间类型。

## 示例

以下示例返回日期 `2023-01-01` 的ISO标准星期几：

```SQL
MySQL > select dayofweek_iso('2023-01-01');
+-----------------------------+
| dayofweek_iso('2023-01-01') |
+-----------------------------+
|                           7 |
+-----------------------------+
```

## 关键词

DAY_OF_WEEK_ISO