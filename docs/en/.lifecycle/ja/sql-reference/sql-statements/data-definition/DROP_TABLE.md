---
displayed_sidebar: "Japanese"
---

# DROP TABLE（テーブルの削除）

## 説明

この文は、テーブルを削除するために使用されます。

## 構文

```sql
DROP TABLE [IF EXISTS] [db_name.]table_name [FORCE]
```

注意：

- テーブルがDROP TABLE文を使用して24時間以内に削除された場合、[RECOVER](../data-definition/RECOVER.md)文を使用してテーブルを復元することができます。
- DROP TABLE FORCEが実行された場合、テーブルは直接削除され、データベース内の未完了のアクティビティがあるかどうかを確認せずに復元することはできません。一般的に、この操作は推奨されません。

## 例

1. テーブルを削除します。

    ```sql
    DROP TABLE my_table;
    ```

2. 存在する場合は、指定されたデータベースのテーブルを削除します。

    ```sql
    DROP TABLE IF EXISTS example_db.my_table;
    ```

3. テーブルを強制的に削除し、ディスク上のデータをクリアします。

    ```sql
    DROP TABLE my_table FORCE;
    ```

## 参考

- [CREATE TABLE（テーブルの作成）](CREATE_TABLE.md)
- [SHOW TABLES（テーブルの表示）](../data-manipulation/SHOW_TABLES.md)
- [SHOW CREATE TABLE（テーブルの作成文の表示）](../data-manipulation/SHOW_CREATE_TABLE.md)
- [ALTER TABLE（テーブルの変更）](ALTER_TABLE.md)
- [SHOW ALTER TABLE（テーブルの変更文の表示）](../data-manipulation/SHOW_ALTER.md)
