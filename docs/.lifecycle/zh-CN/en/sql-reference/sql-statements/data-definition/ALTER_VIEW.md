---
displayed_sidebar: "Chinese"
---

# ALTER VIEW（修改视图）

## 描述

修改视图的定义。

## 语法

```sql
ALTER VIEW
[db_name.]view_name
(column1[ COMMENT "col comment"][, column2, ...])
AS query_stmt
```

注意：

1. 视图是逻辑的，其数据并未存储在物理介质中。在查询时，视图将被用作子查询的一部分。因此，修改视图的定义相当于修改 query_stmt。
2. query_stmt 是任意支持的 SQL。

## 示例

修改 `example_db` 中的 `example_view`。

```sql
ALTER VIEW example_db.example_view
(
c1 COMMENT "列 1",
c2 COMMENT "列 2",
c3 COMMENT "列 3"
)
AS SELECT k1, k2, SUM(v1) 
FROM example_table
GROUP BY k1, k2
```