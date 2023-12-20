---
displayed_sidebar: English
---

# 显示索引

## 描述

该语句用于展示表中索引相关的信息。目前只支持位图索引。

:::提示

执行此操作不需要特定权限。

:::

## 语法

```sql
SHOW INDEX[ES] FROM [db_name.]table_name [FROM database]
Or
SHOW KEY[S] FROM [db_name.]table_name [FROM database]
```

## 示例

1. 展示指定表名（table_name）下的所有索引：

   ```sql
   SHOW INDEX FROM example_db.table_name;
   ```
