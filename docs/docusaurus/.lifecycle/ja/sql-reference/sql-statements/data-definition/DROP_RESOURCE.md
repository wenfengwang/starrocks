```yaml
---
displayed_sidebar: "Japanese"
---

# リソースの削除

## 説明

この文は既存のリソースを削除するために使用されます。ルートユーザーまたはスーパーユーザーのみがリソースを削除できます。

構文:

```sql
DROP RESOURCE 'resource_name'
```

## 例

1. スパークリソースであるspark0を削除します。

    ```SQL
    DROP RESOURCE 'spark0';
    ```

2. Hiveリソースであるhive0を削除します。

    ```SQL
    DROP RESOURCE 'hive0';
    ```