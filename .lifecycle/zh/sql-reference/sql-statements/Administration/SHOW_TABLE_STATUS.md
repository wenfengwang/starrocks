---
displayed_sidebar: English
---

# 显示表状态

## 描述

此语句用于查看表中的部分信息。

:::提示

执行此操作不需要特定的权限。

:::

## 语法

```sql
SHOW TABLE STATUS
[FROM db] [LIKE "pattern"]
```

> 注意
> 该语句主要与MySQL语法兼容。目前，它只显示了一些信息，例如表的注释。

## 示例

1. 查看当前数据库下所有表的信息。

   ```SQL
   SHOW TABLE STATUS;
   ```

2. 查看指定数据库中，名称包含“example”的所有表的信息。

   ```SQL
   SHOW TABLE STATUS FROM db LIKE "%example%";
   ```
