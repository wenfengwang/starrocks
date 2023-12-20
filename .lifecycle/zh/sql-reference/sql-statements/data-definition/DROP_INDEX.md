---
displayed_sidebar: English
---

# 删除索引

## 说明

该语句用于删除表上的特定索引。目前，本版本只支持位图索引。

:::提示

进行此操作需要对目标表有 **ALTER** 权限。您可以根据 [GRANT](../account-management/GRANT.md) 中的指导来授予这项权限。

:::

语法：

```sql
DROP INDEX index_name ON [db_name.]table_name
```
