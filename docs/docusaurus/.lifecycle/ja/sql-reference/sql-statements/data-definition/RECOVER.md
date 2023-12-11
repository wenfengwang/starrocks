---
displayed_sidebar: "Japanese"
---

# RECOVER（復旧）

## 説明

このステートメントは、削除されたデータベース、テーブル、またはパーティションを復旧するために使用されます。

構文：

1. データベースの復旧

    ```sql
    RECOVER DATABASE <db_name>
    ```

2. データベースの復旧

    ```sql
    RECOVER TABLE [<db_name>.]<table_name>
    ```

3. パーティションの復旧

    ```sql
    RECOVER PARTITION partition_name FROM [<db_name>.]<table_name>
    ```

注意：

1. 以前に削除されたメタ情報のみを復旧できます。デフォルトの時間は1日です（fe.confのパラメーター設定catalog_trash_expire_secondを通じて変更できます）。
2. 同一のメタ情報が作成された状態で削除された場合、以前のメタ情報は復旧できません。

## 例

1. example_dbという名前のデータベースを復旧する

    ```sql
    RECOVER DATABASE example_db;
    ```

2. テーブル名がexample_tblのテーブルを復旧する

    ```sql
    RECOVER TABLE example_db.example_tbl;
    ```

3. example_tblテーブルでp1という名前のパーティションを復旧する

    ```sql
    RECOVER PARTITION p1 FROM example_tbl;
    ```