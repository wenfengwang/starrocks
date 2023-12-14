---
displayed_sidebar: "Chinese"
---

# VARCHAR

## 描述

VARCHAR(M)

Variable length string. `M` represents the length of the variable length string, unit: byte, default value is `1`.

- Prior to StarRocks 2.1, the value range of `M` is 1~65533.
- 【In public beta】Starting from StarRocks 2.1, the value range of `M` is 1~1048576.

## 示例

Specify the field type as VARCHAR when creating a table.

```sql
CREATE TABLE varcharDemo (
    pk INT COMMENT "range [-2147483648, 2147483647]",
    pd_type VARCHAR(20) COMMENT "range char(m),m in (1-65533) "
) ENGINE=OLAP 
DUPLICATE KEY(pk)
COMMENT "OLAP"
DISTRIBUTED BY HASH(pk);
```