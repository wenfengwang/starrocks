---
displayed_sidebar: "中文"
---

# DATETIME

## 描述

日期时间类型的取值范围为 ['0000-01-01 00:00:00', '9999-12-31 23:59:59']。

打印格式为'YYYY-MM-DD HH:MM:SS'

## 示例

创建表时，指定某字段的类型为 DATETIME。

```sql
CREATE TABLE dateTimeDemo (
    pk INT COMMENT "范围 [-2147483648, 2147483647]",
    relTime DATETIME COMMENT "YYYY-MM-DD HH:MM:SS"
) ENGINE=OLAP 
DUPLICATE KEY(pk)
COMMENT "OLAP"
DISTRIBUTED BY HASH(pk);
```