---
displayed_sidebar: English
---

# 删除视图

## 描述

此语句用于删除一个逻辑视图 VIEW

## 语法

```sql
DROP VIEW [IF EXISTS]
[db_name.]view_name
```

## 示例

1. 如果存在，那么删除 example_db 中的视图 example_view。

   ```sql
   DROP VIEW IF EXISTS example_db.example_view;
   ```