---
displayed_sidebar: "中文"
---

# 日期

## 描述

日期（DATE）类型，其有效范围目前为 ['0000-01-01', '9999-12-31']，默认的显示格式是 'YYYY-MM-DD'。

## 示例

在创建表时，指定某个字段的类型为 DATE。

```sql
CREATE TABLE dateDemo (
    pk INT COMMENT "范围 [-2147483648, 2147483647]",
    make_time DATE NOT NULL COMMENT "YYYY-MM-DD"
) ENGINE=OLAP 
DUPLICATE KEY(pk)
COMMENT "OLAP"
DISTRIBUTED BY HASH(pk);
```