---
displayed_sidebar: English
---

# 删除资源

## 描述

此语句用于删除已存在的资源。只有 root 或超级用户可以删除资源。

语法：

```sql
DROP RESOURCE 'resource_name'
```

## 示例

1. 删除名为 spark0 的 Spark 资源。

   ```SQL
   DROP RESOURCE 'spark0';
   ```

2. 删除名为 hive0 的 Hive 资源。

   ```SQL
   DROP RESOURCE 'hive0';
   ```