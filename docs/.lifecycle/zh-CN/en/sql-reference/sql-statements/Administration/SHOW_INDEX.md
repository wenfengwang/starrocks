---
displayed_sidebar: "Chinese"
---

# 显示索引

## 描述

此语句用于显示表中与索引相关的信息。目前仅支持位图索引。

语法:

```sql
SHOW INDEX[ES] FROM [db_name.]table_name [FROM database]
或
SHOW KEY[S] FROM [db_name.]table_name [FROM database]
```

## 例子

1. 显示指定表名下的所有索引：

    ```sql
    SHOW INDEX FROM example_db.table_name;
    ```