---
displayed_sidebar: Chinese
---

# DROP DATABASE

## 機能

このステートメントはデータベースを削除するために使用されます。

> **注意**
>
> この操作を行うには、対象のデータベースに対するDROP権限が必要です。

## 文法

```sql
-- バージョン2.0以前
DROP DATABASE [IF EXISTS] [FORCE] db_name
-- バージョン2.0以降
DROP DATABASE [IF EXISTS] db_name [FORCE]
```

以下の点に注意してください：

- DROP DATABASEを実行した後、一定期間（デフォルトは1日）内に、[RECOVER](../data-definition/RECOVER.md) ステートメントを使用して削除されたデータベースを復元することができますが、データベースと一緒に削除されたPipeインポートジョブ（バージョン3.2以降でサポート）は復元できません。
- `DROP DATABASE FORCE`を実行すると、システムはデータベースに未完了のトランザクションがあるかどうかをチェックせず、データベースは直接削除され、**復元できません**。通常、この操作は推奨されません。
- データベースを削除すると、そのデータベースに属するすべてのPipeインポートジョブ（バージョン3.2以降でサポート）も削除されます。

## 例

1. データベース db_test を削除します。

    ```sql
    DROP DATABASE db_test;
    ```

## 関連リファレンス

- [CREATE DATABASE](../data-definition/CREATE_DATABASE.md)
- [SHOW CREATE DATABASE](../data-manipulation/SHOW_CREATE_DATABASE.md)
- [USE](../data-definition/USE.md)
- [DESCRIBE](../Utility/DESCRIBE.md)
- [SHOW DATABASES](../data-manipulation/SHOW_DATABASES.md)
