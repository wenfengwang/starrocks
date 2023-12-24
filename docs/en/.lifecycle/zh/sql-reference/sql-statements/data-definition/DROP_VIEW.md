---
displayed_sidebar: English
---

# 删除视图

## 描述

此语句用于删除逻辑视图 VIEW

## 语法

```sql
DROP VIEW [IF EXISTS]
[db_name.]view_name
```

## 例子

1. 如果存在，则删除example_db上的example_view视图。

    ```sql
    DROP VIEW IF EXISTS example_db.example_view;
    ```
