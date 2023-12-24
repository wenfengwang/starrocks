---
displayed_sidebar: English
---

# 删除索引

## 描述

此语句用于删除表上的指定索引。目前，此版本仅支持位图索引。

:::提示

此操作需要对目标表具有 ALTER 权限。您可以按照 [GRANT](../account-management/GRANT.md) 中的说明授予此权限。

:::

语法：

```sql
DROP INDEX index_name ON [db_name.]table_name