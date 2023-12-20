---
displayed_sidebar: English
---

# 创建索引

## 描述

此语句用于创建索引。

:::tip

此操作需要目标表的 **ALTER** 权限。您可以按照 [GRANT](../account-management/GRANT.md) 中的说明来授予此权限。

:::

## 语法

```sql
CREATE INDEX index_name ON table_name (column [, ...],) [USING BITMAP] [COMMENT 'balabala']
```

注意：

1. 当前版本只支持位图索引。
2. 只能在单一列上创建 BITMAP 索引。

## 示例

1. 在 `table1` 上为 `siteid` 创建位图索引。

   ```sql
   CREATE INDEX index_name ON table1 (siteid) USING BITMAP COMMENT 'balabala';
   ```