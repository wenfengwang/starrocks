---
displayed_sidebar: "Japanese"
---

# リソースの削除

## 説明

この文は、既存のリソースを削除するために使用されます。ルートユーザーまたはスーパーユーザーのみがリソースを削除できます。

構文:

```sql
DROP RESOURCE 'resource_name'
```

## 例

1. spark0という名前のSparkリソースを削除します。

    ```SQL   
    DROP RESOURCE 'spark0';
    ```

2. hive0という名前のHiveリソースを削除します。

    ```SQL   
    DROP RESOURCE 'hive0';
    ```