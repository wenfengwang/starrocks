---
displayed_sidebar: English
---

# DROP DATABASE

## 説明

StarRocksでデータベースを削除します。

> **注**
>
> この操作には、対象データベースに対するDROP権限が必要です。

## 構文

```sql
DROP DATABASE [IF EXISTS] <db_name> [FORCE]
```

以下の点に注意してください：

- DROP DATABASEを実行してデータベースを削除した後、指定された保持期間（デフォルトの保持期間は1日）内に[RECOVER](../data-definition/RECOVER.md)文を使用して削除されたデータベースを復元できますが、データベースと共に削除されたパイプ（v3.2以降でサポートされています）は復元できません。
- `DROP DATABASE FORCE`を実行してデータベースを削除する場合、データベースは直接削除され、中に未完了のアクティビティがあるかどうかのチェックは行われず、復元はできません。通常、この操作は推奨されません。
- データベースを削除すると、そのデータベースに属するすべてのパイプ（v3.2以降でサポートされています）もデータベースと共に削除されます。

## 例

1. データベースdb_testを削除します。

    ```sql
    DROP DATABASE db_test;
    ```

## 参照

- [CREATE DATABASE](../data-definition/CREATE_DATABASE.md)
- [SHOW CREATE DATABASE](../data-manipulation/SHOW_CREATE_DATABASE.md)
- [USE](../data-definition/USE.md)
- [DESCRIBE](../Utility/DESCRIBE.md)
- [SHOW DATABASES](../data-manipulation/SHOW_DATABASES.md)
