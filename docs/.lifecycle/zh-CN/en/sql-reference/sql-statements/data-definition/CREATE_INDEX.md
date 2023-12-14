---
displayed_sidebar: "English"
---

# 创建索引

## 描述

该语句用于创建索引。

语法：

```sql
CREATE INDEX index_name ON table_name (column [, ...],) [USING BITMAP] [COMMENT '备注']
```

注意：

1. 当前版本仅支持位图索引。
2. 只能在单个列上创建位图索引。

## 示例

1. 为 `table1` 的 `siteid` 创建位图索引。

    ```sql
    CREATE INDEX index_name ON table1 (siteid) USING BITMAP COMMENT '备注';
    ```