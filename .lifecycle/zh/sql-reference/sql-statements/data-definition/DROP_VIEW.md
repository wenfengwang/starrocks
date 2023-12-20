---
displayed_sidebar: English
---

# 删除视图

## 描述

此语句用于删除一个逻辑视图VIEW。

## 语法

```sql
DROP VIEW [IF EXISTS]
[db_name.]view_name
```

## 示例

1. 如果存在，则从example_db中删除视图example_view。

   ```sql
   DROP VIEW IF EXISTS example_db.example_view;
   ```
