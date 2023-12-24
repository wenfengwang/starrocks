---
displayed_sidebar: English
---

# 日期

## 描述

DATE 类型。当前值范围为['0000-01-01','9999-12-31']，默认格式为 `YYYY-MM-DD`。

## 例子

示例 1：在创建表时将某一列指定为 DATE 类型。

```SQL
CREATE TABLE dateDemo (
    pk INT COMMENT "范围为[-2147483648, 2147483647]",
    make_time DATE NOT NULL COMMENT "YYYY-MM-DD"
) ENGINE=OLAP 
DUPLICATE KEY(pk)
COMMENT "OLAP"
DISTRIBUTED BY HASH(pk)
```

示例 2：将 DATETIME 值转换为 DATE 值。

```sql
mysql> SELECT DATE('2003-12-31 01:02:03');
-> '2003-12-31'
```

有关更多信息，请参阅 [date](../../sql-functions/date-time-functions/date.md) 函数。
