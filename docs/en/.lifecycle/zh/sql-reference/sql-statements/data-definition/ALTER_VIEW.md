---
displayed_sidebar: English
---

# 修改视图

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

1. 视图是逻辑的，其中的数据并不存储在物理介质中。当查询时，该视图将被用作语句中的子查询。因此，修改视图的定义等同于修改 query_stmt。
2. query_stmt 是任意支持的 SQL。

## 例子

修改 `example_db` 上的 `example_view`。

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