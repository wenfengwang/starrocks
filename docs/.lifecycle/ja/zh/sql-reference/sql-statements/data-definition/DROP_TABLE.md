---
displayed_sidebar: Chinese
---

# DROP TABLE

## 機能

このステートメントはテーブルを削除するために使用されます。

## 文法

```sql
DROP TABLE [IF EXISTS] [db_name.]table_name [FORCE]
```

説明：

1. `DROP TABLE` を実行した後、一定期間（デフォルトは1日）内に、[RECOVER](../data-definition/RECOVER.md) ステートメントを使用して削除されたテーブルを復元できます。RECOVER ステートメントの詳細についてはこちらをご覧ください。

2. `DROP TABLE FORCE` を実行すると、システムはテーブルに未完了のトランザクションが存在するかどうかをチェックせず、テーブルは直接削除され、復元することはできません。通常、この操作は推奨されません。

## 例

1. テーブルを削除する。

    ```sql
    DROP TABLE my_table;
    ```

2. 存在する場合、指定されたデータベースのテーブルを削除する。

    ```sql
    DROP TABLE IF EXISTS example_db.my_table;
    ```

3. テーブルを強制的に削除し、ディスク上のファイルをクリーンアップする。

    ```sql
    DROP TABLE my_table FORCE;
    ```

## 参照

* [CREATE TABLE](CREATE_TABLE.md)
* [SHOW TABLES](../data-manipulation/SHOW_TABLES.md)
* [SHOW CREATE TABLE](../data-manipulation/SHOW_CREATE_TABLE.md)
* [ALTER TABLE](ALTER_TABLE.md)
* [SHOW ALTER TABLE](../data-manipulation/SHOW_ALTER.md)
