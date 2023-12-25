---
displayed_sidebar: English
---

# CREATE DATABASE

## 説明

このステートメントは、データベースを作成するために使用されます。

:::tip

この操作には、対象カタログに対するCREATE DATABASE権限が必要です。この権限を付与するには、[GRANT](../account-management/GRANT.md)の説明に従ってください。

:::

## 構文

```sql
CREATE DATABASE [IF NOT EXISTS] <db_name>
```

## 例

1. `db_test` データベースを作成する。

    ```sql
    CREATE DATABASE db_test;
    ```

## 参照

- [SHOW CREATE DATABASE](../data-manipulation/SHOW_CREATE_DATABASE.md)
- [USE](../data-definition/USE.md)
- [SHOW DATABASES](../data-manipulation/SHOW_DATABASES.md)
- [DESC](../Utility/DESCRIBE.md)
- [DROP DATABASE](../data-definition/DROP_DATABASE.md)
