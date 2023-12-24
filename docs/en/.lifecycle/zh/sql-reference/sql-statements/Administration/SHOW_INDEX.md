---
displayed_sidebar: English
---

# 显示索引

## 描述

该语句用于显示表中索引的相关信息。目前仅支持位图索引。

:::提示

此操作无需权限。

:::

## 语法

```sql
SHOW INDEX[ES] FROM [db_name.]table_name [FROM database]
或
SHOW KEY[S] FROM [db_name.]table_name [FROM database]
```

## 例子

1. 显示指定table_name下的所有索引：

    ```sql
    SHOW INDEX FROM example_db.table_name;
    ```
