---
displayed_sidebar: "Chinese"
---

# 浮点数

## 描述

4 字节浮点数。

## 示例

创建表时指定字段类型为浮点数。

```sql
CREATE TABLE floatDemo (
    pk BIGINT(20) NOT NULL COMMENT "",
    channel FLOAT COMMENT "4 字节"
) ENGINE=OLAP 
DUPLICATE KEY(pk)
COMMENT "OLAP"
DISTRIBUTED BY HASH(pk);
```