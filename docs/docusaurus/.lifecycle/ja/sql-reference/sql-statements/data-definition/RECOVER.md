---
displayed_sidebar: "Japanese"
---

# RECOVER（復元）

## 説明

このステートメントは、削除されたデータベース、テーブル、またはパーティションを復元するために使用されます。

構文:

1. データベースを復元する

    ```sql
    RECOVER DATABASE <db_name>
    ```

2. データベースを復元する

    ```sql
    RECOVER TABLE [<db_name>.]<table_name>
    ```

3. パーティションを復元する

    ```sql
    RECOVER PARTITION partition_name FROM [<db_name>.]<table_name>
    ```

注意:

1. 以前に削除されたメタ情報のみを復元できます。デフォルトの時間は1日です。（fe.confのcatalog_trash_expire_secondパラメータの設定を通じて変更できます。）
2. 同じメタ情報で削除と作成が行われた場合、以前のメタ情報は復元されません。

## 例

1. example_dbという名前のデータベースを復元する

    ```sql
    RECOVER DATABASE example_db;
    ```

2. example_tblという名前のテーブルを復元する

    ```sql
    RECOVER TABLE example_db.example_tbl;
    ```

3. example_tblテーブル内のp1という名前のパーティションを復元する

    ```sql
    RECOVER PARTITION p1 FROM example_tbl;
    ```