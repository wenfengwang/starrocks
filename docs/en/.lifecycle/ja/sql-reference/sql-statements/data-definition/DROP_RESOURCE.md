---
displayed_sidebar: "Japanese"
---

# リソースの削除

## 説明

このステートメントは、既存のリソースを削除するために使用されます。ルートまたはスーパーユーザーのみがリソースを削除できます。

構文:

```sql
DROP RESOURCE 'リソース名'
```

## 例

1. spark0 という名前の Spark リソースを削除する。

    ```SQL
    DROP RESOURCE 'spark0';
    ```

2. hive0 という名前の Hive リソースを削除する。

    ```SQL
    DROP RESOURCE 'hive0';
    ```
