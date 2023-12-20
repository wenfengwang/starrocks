---
displayed_sidebar: English
---

# SHOW FULL COLUMNS

## 描述

此语句用于展示指定表中各列的详细信息。

:::tip

此操作不需要特定权限。

:::

## 语法

```sql
SHOW FULL COLUMNS FROM <tbl_name>
```

## 示例

1. 查看指定表中的列详细信息。

   ```sql
   SHOW FULL COLUMNS FROM tbl;
   ```