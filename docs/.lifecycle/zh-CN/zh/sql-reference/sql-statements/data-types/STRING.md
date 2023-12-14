---
displayed_sidebar: "Chinese"
---

# STRING

## 描述

字符串，最大长度 65533 字节。

## 示例

创建表时指定字段类型为 STRING。

```sql
CREATE TABLE stringDemo (
    pk INT COMMENT "范围 [-2147483648, 2147483647]",
    us_detail STRING COMMENT "上限值 65533 字节"
) ENGINE=OLAP 
DUPLICATE KEY(pk)
COMMENT "OLAP"
DISTRIBUTED BY HASH(pk);
```

表建成功后通过执行 `desc <table_name>;` 查看表信息，可以看到 STRING 类型为 `VARCHAR(65533)`。