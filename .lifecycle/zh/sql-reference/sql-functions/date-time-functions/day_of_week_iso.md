---
displayed_sidebar: English
---

# ISO星期几

## 描述

返回指定日期对应的ISO标准星期几，结果为1到7之间的整数。根据该标准，1代表星期一，7代表星期日。

## 语法

```Haskell
INT DAY_OF_WEEK_ISO(DATETIME date)
```

## 参数

date：需要转换的日期。该参数必须为DATE或DATETIME类型。

## 示例

以下示例返回2023-01-01这一日期对应的ISO标准星期几：

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
