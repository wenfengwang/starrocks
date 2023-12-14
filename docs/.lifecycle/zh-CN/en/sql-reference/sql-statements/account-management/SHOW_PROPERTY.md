---
displayed_sidebar: "Chinese"
---

# 显示属性

## 描述

此语句用于查看用户的属性。

语法：

```sql
SHOW PROPERTY [FOR user] [LIKE key]
```

## 例子

1. 查看jack用户的属性

    ```sql
    SHOW PROPERTY FOR 'jack'
    ```

2. 查看与由Jack用户导入的集群相关的属性

    ```sql
    SHOW PROPERTY FOR 'jack' LIKE '%load_cluster%'
    ```