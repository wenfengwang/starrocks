---
displayed_sidebar: "中文"
---

# 布尔值

## 描述

BOOL, BOOLEAN

与 TINYINT 相同，0 表示 false，1 表示 true。

## 示例

在创建表时指定字段类型为 BOOLEAN。

```sql
CREATE TABLE booleanDemo (
    pk INT COMMENT "范围 [-2147483648, 2147483647]",
    ispass BOOLEAN COMMENT "true/false"
) ENGINE=OLAP 
DUPLICATE KEY(pk)
COMMENT "OLAP"
DISTRIBUTED BY HASH(pk);
```