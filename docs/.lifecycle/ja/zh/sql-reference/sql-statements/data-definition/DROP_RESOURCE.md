---
displayed_sidebar: Chinese
---

# DROP RESOURCE

## 機能

このステートメントは、既存のリソースを削除するために使用されます。リソースを削除するには、DROP権限を持っている必要があります。

RESOURCE の作成については、[CREATE RESOURCE](../data-definition/CREATE_RESOURCE.md) の章を参照してください。

## 文法

```sql
DROP RESOURCE 'resource_name'
```

## 例

1. 名前が spark0 の Spark リソースを削除します。

    ```SQL
    DROP RESOURCE 'spark0';
    ```

2. 名前が hive0 の Hive リソースを削除します。

    ```SQL
    DROP RESOURCE 'hive0';
    ```
