---
displayed_sidebar: "Japanese"
---

# データベースの削除

## 説明

StarRocks でデータベースを削除します。

> **注記**
>
> この操作には、宛先データベースでのDROP 権限が必要です。

## 構文

```sql
DROP DATABASE [IF EXISTS] <db_name> [FORCE]
```

以下の点に注意してください:

- データベースを削除するためにDROP DATABASEを実行した後、指定された保存期間内（デフォルトの保存期間は1日です）に[RECOVER](../data-definition/RECOVER.md)ステートメントを使用して削除されたデータベースを復元できますが、データベースと一緒に削除されたパイプ（v3.2以降でサポート）を復元することはできません。
- `DROP DATABASE FORCE`を実行してデータベースを削除すると、未完了のアクティビティがあるかどうかをチェックせずにデータベースが直接削除され、復元できません。一般的に、この操作は推奨されません。
- データベースを削除すると、そのデータベースに属するすべてのパイプ（v3.2以降でサポート）が削除されます。

## 例

1. データベース db_test を削除します。

    ```sql
    DROP DATABASE db_test;
    ```

## 参照

- [CREATE DATABASE](../data-definition/CREATE_DATABASE.md)
- [SHOW CREATE DATABASE](../data-manipulation/SHOW_CREATE_DATABASE.md)
- [USE](../data-definition/USE.md)
- [DESC](../Utility/DESCRIBE.md)
- [SHOW DATABASES](../data-manipulation/SHOW_DATABASES.md)