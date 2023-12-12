---
displayed_sidebar: "Japanese"
---

# データベースを削除する

## 説明

StarRocksでデータベースを削除します。

> **注意**
>
> この操作には、宛先データベースのDROP権限が必要です。

## 構文

```sql
DROP DATABASE [IF EXISTS] <db_name> [FORCE]
```

次の点に注意してください。

- データベースを削除すると、指定された保持期間（デフォルトの保持期間は1日）内に[RECOVER](../data-definition/RECOVER.md)ステートメントを使用して削除されたデータベースを復元できます（v3.2以降でサポートされているパイプは復元できません）。
- `DROP DATABASE FORCE`を実行してデータベースを削除すると、未完了のアクティビティがあるかどうかをチェックせずにデータベースが直接削除され、復元できません。一般的に、この操作は推奨されません。
- データベースを削除すると、そのデータベースに属するすべてのパイプ（v3.2以降でサポートされています）が削除されます。

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