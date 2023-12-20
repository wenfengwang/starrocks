---
displayed_sidebar: English
---

# SHOW INDEX

## 描述

此语句用于展示表中索引相关的信息。目前只支持位图索引。

:::tip

此操作不需要权限。

:::

## 语法

```sql
SHOW INDEX[ES] FROM [db_name.]table_name [FROM database]
或
SHOW KEY[S] FROM [db_name.]table_name [FROM database]
```

## 示例

1. 显示指定table_name下的所有索引：

   ```sql
   SHOW INDEX FROM example_db.table_name;
   ```