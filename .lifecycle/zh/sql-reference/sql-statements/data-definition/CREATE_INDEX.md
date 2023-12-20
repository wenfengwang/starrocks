---
displayed_sidebar: English
---

# 创建索引

## 说明

此语句用于创建索引。

:::提示

执行此操作需要对目标表拥有**ALTER**权限。您可以遵循[GRANT](../account-management/GRANT.md)语句中的指引来授予相应权限。

:::

## 语法

```sql
CREATE INDEX index_name ON table_name (column [, ...],) [USING BITMAP] [COMMENT'balabala']
```

注意：

1. 当前版本只支持位图索引。
2. 只能在单一列上创建位图索引。

## 示例

1. 为table1表中的siteid列创建位图索引。

   ```sql
   CREATE INDEX index_name ON table1 (siteid) USING BITMAP COMMENT 'balabala';
   ```
