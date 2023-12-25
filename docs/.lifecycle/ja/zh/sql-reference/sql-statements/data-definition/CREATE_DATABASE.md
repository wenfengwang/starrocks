---
displayed_sidebar: Chinese
---

# CREATE DATABASE

## 機能

このステートメントはデータベース（database）を作成するために使用されます。

:::tip

この操作には対応するCatalogのCREATE DATABASE権限が必要です。ユーザーに権限を付与するには[GRANT](../account-management/GRANT.md)を参照してください。

:::

## 文法

```sql
CREATE DATABASE [IF NOT EXISTS] db_name
```

## 例

1. 新しいデータベース db_test を作成します。

    ```sql
    CREATE DATABASE db_test;
    ```

## 関連リファレンス

- [SHOW CREATE DATABASE](../data-manipulation/SHOW_CREATE_DATABASE.md)
- [USE](../data-definition/USE.md)
- [SHOW DATABASES](../data-manipulation/SHOW_DATABASES.md)
- [DESCRIBE](../Utility/DESCRIBE.md)
- [DROP DATABASE](../data-definition/DROP_DATABASE.md)
