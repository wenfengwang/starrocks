---
displayed_sidebar: English
---

# 日期

## 说明

日期类型。目前的值域是['0000-01-01','9999-12-31']，默认的格式是YYYY-MM-DD。

## 示例

示例1：在创建表时，将某一列指定为日期（DATE）类型。

```SQL
CREATE TABLE dateDemo (
    pk INT COMMENT "range [-2147483648, 2147483647]",
    make_time DATE NOT NULL COMMENT "YYYY-MM-DD"
) ENGINE=OLAP 
DUPLICATE KEY(pk)
COMMENT "OLAP"
DISTRIBUTED BY HASH(pk)
```

示例2：将日期时间（DATETIME）值转换为日期（DATE）值。

```sql
mysql> SELECT DATE('2003-12-31 01:02:03');
-> '2003-12-31'
```

更多信息，请参考[date](../../sql-functions/date-time-functions/date.md)函数。
