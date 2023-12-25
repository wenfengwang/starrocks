---
displayed_sidebar: English
---

# RECOVER

## 説明

このステートメントは、削除されたデータベース、テーブル、またはパーティションを復旧するために使用されます。

構文：

1. データベースの復旧

    ```sql
    RECOVER DATABASE <db_name>
    ```

2. テーブルの復旧

    ```sql
    RECOVER TABLE [<db_name>.]<table_name>
    ```

3. パーティションの復旧

    ```sql
    RECOVER PARTITION partition_name FROM [<db_name>.]<table_name>
    ```

注意：

1. 少し前に削除されたメタ情報のみを復元できます。デフォルトの時間は1日です。(fe.confのパラメータ設定catalog_trash_expire_secondで変更できます。)
2. 同一のメタ情報を作成した後にメタ情報を削除した場合、以前のメタ情報は復元されません。

## 例

1. example_dbという名前のデータベースを復旧する

    ```sql
    RECOVER DATABASE example_db;
    ```

2. example_tblという名前のテーブルを復旧する

    ```sql
    RECOVER TABLE example_db.example_tbl;
    ```

3. example_tblテーブル内のp1という名前のパーティションを復旧する

    ```sql
    RECOVER PARTITION p1 FROM example_tbl;
    ```
