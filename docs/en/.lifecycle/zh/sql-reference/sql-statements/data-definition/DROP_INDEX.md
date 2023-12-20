---
displayed_sidebar: English
---

# 删除索引

## 描述

此语句用于删除表上的指定索引。当前版本仅支持位图索引。

:::tip

此操作需要对目标表的 ALTER 权限。您可以按照 [GRANT](../account-management/GRANT.md) 文档中的指示来授予此权限。

:::

语法：

```sql
DROP INDEX index_name ON [db_name.]table_name
```