---
displayed_sidebar: English
---

# ALTER VIEW

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

1. 视图是逻辑上的，其中数据并不存储在物理介质上。查询时，视图会作为语句中的子查询被使用。因此，修改视图的定义等同于修改query_stmt。
2. query_stmt 支持任意SQL语句。

## 示例

修改 `example_db` 中的 `example_view`。

```sql
ALTER VIEW example_db.example_view
(
c1 COMMENT "column 1",
c2 COMMENT "column 2",
c3 COMMENT "column 3"
)
AS SELECT k1, k2, SUM(v1) 
FROM example_table
GROUP BY k1, k2
```