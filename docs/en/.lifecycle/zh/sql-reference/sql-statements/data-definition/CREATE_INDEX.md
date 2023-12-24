---
displayed_sidebar: English
---

# 创建索引

## 描述

此语句用于创建索引。

:::提示
此操作需要对目标表具有 ALTER 权限。您可以按照 [GRANT](../account-management/GRANT.md) 中的说明授予此权限。
:::

## 语法

```sql
CREATE INDEX index_name ON table_name (column [, ...]) [USING BITMAP] [COMMENT 'balabala']
```

注意：

1. 仅支持当前版本中的位图索引。
2. 仅能在单个列中创建 BITMAP 索引。

## 例子

1. 为 `table1` 的 `siteid` 创建位图索引。

    ```sql
    CREATE INDEX index_name ON table1 (siteid) USING BITMAP COMMENT 'balabala';
    ```