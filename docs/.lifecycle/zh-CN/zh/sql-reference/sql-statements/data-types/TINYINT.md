---
displayed_sidebar: "中文"
---

# TINYINT

## 描述

1 字节有符号整数，范围 [-128, 127]。

## 示例

创建表时指定字段类型为 TINYINT。

```sql
CREATE TABLE tinyIntDemo (
    pk TINYINT COMMENT "范围 [-128, 127]",
    pd_type VARCHAR(20) COMMENT "范围 char(m)，m 在 (1-255)之间 "
) ENGINE=OLAP 
DUPLICATE KEY(pk)
COMMENT "OLAP"
DISTRIBUTED BY HASH(pk);
```