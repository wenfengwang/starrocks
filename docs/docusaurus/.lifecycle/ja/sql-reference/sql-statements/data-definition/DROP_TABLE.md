---
displayed_sidebar: "Japanese"
---

# DROP TABLE

## 説明

このステートメントは、テーブルを削除するために使用されます。

## 構文

```sql
DROP TABLE [IF EXISTS] [db_name.]table_name [FORCE]
```

注意：

- DROP TABLE ステートメントを使用して24時間以内にテーブルが削除された場合、[RECOVER](../data-definition/RECOVER.md) ステートメントを使用してテーブルを復元できます。
- DROP TABLE FORCE が実行された場合、テーブルは直接削除され、データベース内に未完了のアクティビティがあるかどうかを確認せずに復旧することはできません。一般的に、この操作は推奨されません。

## 例

1. テーブルを削除します。

    ```sql
    DROP TABLE my_table;
    ```

2. 存在する場合は、指定されたデータベース上のテーブルを削除します。

    ```sql
    DROP TABLE IF EXISTS example_db.my_table;
    ```

3. テーブルを強制的に削除し、ディスク上のデータをクリアします。

    ```sql
    DROP TABLE my_table FORCE;
    ```

## 参照

- [CREATE TABLE](CREATE_TABLE.md)
- [SHOW TABLES](../data-manipulation/SHOW_TABLES.md)
- [SHOW CREATE TABLE](../data-manipulation/SHOW_CREATE_TABLE.md)
- [ALTER TABLE](ALTER_TABLE.md)
- [SHOW ALTER TABLE](../data-manipulation/SHOW_ALTER.md)