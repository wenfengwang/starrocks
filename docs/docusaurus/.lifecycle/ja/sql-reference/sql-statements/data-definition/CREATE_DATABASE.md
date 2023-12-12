---
displayed_sidebar: "Japanese"
---

# データベースの作成

## 説明

この文はデータベースを作成するために使用されます。

## 構文

```sql
CREATE DATABASE [IF NOT EXISTS] <db_name>
```

## 例

1. `db_test`というデータベースを作成します。

    ```sql
    CREATE DATABASE db_test;
    ```

## 参照

- [SHOW CREATE DATABASE](../data-manipulation/SHOW_CREATE_DATABASE.md)
- [USE](../data-definition/USE.md)
- [SHOW DATABASES](../data-manipulation/SHOW_DATABASES.md)
- [DESC](../Utility/DESCRIBE.md)
- [DROP DATABASE](../data-definition/DROP_DATABASE.md)