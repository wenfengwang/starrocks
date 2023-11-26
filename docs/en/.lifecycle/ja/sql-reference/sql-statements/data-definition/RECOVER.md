---
displayed_sidebar: "Japanese"
---

# RECOVER（復旧）

## 説明

このステートメントは、削除されたデータベース、テーブル、またはパーティションを復旧するために使用されます。

構文:

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

注意:

1. 一定時間前に削除されたメタ情報のみを復旧できます。デフォルトの時間は1日です。（fe.confのcatalog_trash_expire_secondパラメータの設定で変更できます。）
2. 同じメタ情報が作成されている場合、以前のメタ情報は復旧されません。

## 例

1. example_dbという名前のデータベースを復旧する

    ```sql
    RECOVER DATABASE example_db;
    ```

2. example_tblという名前のテーブルを復旧する

    ```sql
    RECOVER TABLE example_db.example_tbl;
    ```

3. example_tblテーブルのp1という名前のパーティションを復旧する

    ```sql
    RECOVER PARTITION p1 FROM example_tbl;
    ```
