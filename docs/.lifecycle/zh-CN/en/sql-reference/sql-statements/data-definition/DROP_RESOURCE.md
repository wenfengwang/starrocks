---
displayed_sidebar: "Chinese"
---

# 删除资源

## 描述

此语句用于删除现有的资源。只有root或超级用户才能删除资源。

语法:

```sql
DROP RESOURCE '资源名称'
```

## 例子

1. 删除名为spark0的Spark资源。

    ```SQL
    DROP RESOURCE 'spark0';
    ```

2. 删除名为hive0的Hive资源。

    ```SQL
    DROP RESOURCE 'hive0';
    ```