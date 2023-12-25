---
displayed_sidebar: English
---

# DROP RESOURCE

## 説明

このステートメントは、既存のリソースを削除するために使用されます。rootまたはsuperuserのみがリソースを削除できます。

構文：

```sql
DROP RESOURCE 'resource_name'
```

## 例

1. spark0 という名前のSparkリソースを削除します。

    ```SQL
    DROP RESOURCE 'spark0';
    ```

2. hive0 という名前のHiveリソースを削除します。

    ```SQL
    DROP RESOURCE 'hive0';
    ```
