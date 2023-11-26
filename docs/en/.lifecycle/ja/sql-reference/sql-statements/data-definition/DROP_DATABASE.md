---
displayed_sidebar: "Japanese"
---

# データベースの削除

## 説明

StarRocksでデータベースを削除します。

> **注意**
>
> この操作には、宛先データベースでのDROP権限が必要です。

## 構文

```sql
DROP DATABASE [IF EXISTS] <db_name> [FORCE]
```

注意：

1. DROP DATABASEを実行した後、しばらくすると、RECOVERステートメントを使用して削除されたデータベースを復元することができます。詳細については、RECOVERステートメントを参照してください。
2. DROP DATABASE FORCEを実行すると、データベースは直接削除され、データベース内に未完了のアクティビティがあるかどうかを確認せずに復元することはできません。通常、この操作は推奨されません。

## 例

1. データベースdb_testを削除します。

    ```sql
    DROP DATABASE db_test;
    ```

## 参照

- [CREATE DATABASE](../data-definition/CREATE_DATABASE.md)
- [SHOW CREATE DATABASE](../data-manipulation/SHOW_CREATE_DATABASE.md)
- [USE](../data-definition/USE.md)
- [DESC](../Utility/DESCRIBE.md)
- [SHOW DATABASES](../data-manipulation/SHOW_DATABASES.md)
