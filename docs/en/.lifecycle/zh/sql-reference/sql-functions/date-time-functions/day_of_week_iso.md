---
displayed_sidebar: English
---

# dayofweek_iso

## 描述

返回指定日期的 ISO 标准星期几，结果为 `1` 到 `7` 范围内的整数。在此标准中，`1` 代表星期一，`7` 代表星期日。

## 语法

```Haskell
INT DAY_OF_WEEK_ISO(DATETIME date)
```

## 参数

`date`：您要转换的日期。它必须是 DATE 或 DATETIME 类型。

## 示例

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