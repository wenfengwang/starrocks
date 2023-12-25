---
displayed_sidebar: Chinese
---

# ALTER TABLE のキャンセル

## 機能

進行中の ALTER TABLE 操作をキャンセルすることができます。これには以下が含まれます:

- 列の変更。
- テーブル構造の最適化（バージョン3.2以降）、バケット方式とバケット数の変更を含む。
- Rollup インデックスの作成と削除。

> **注意**
>
> - このステートメントは同期操作です。
> - ALTER 権限を持つユーザーのみがこのステートメントを実行できます。
> - このステートメントは、ALTER TABLE で実行される非同期操作のみをキャンセルすることができます（上記のように）。ALTER TABLE で実行される同期操作（例えば rename）はキャンセルできません。

## 文法

   ```SQL
   CANCEL ALTER TABLE { COLUMN | OPTIMIZE | ROLLUP } FROM [<db_name>.]<table_name>
   ```

## パラメータ説明

- `{COLUMN | OPTIMIZE | ROLLUP}`：`COLUMN`、`OPTIMIZE`、`ROLLUP` の中から一つを選択する必要があります。
  - `COLUMN` を指定した場合、列の変更操作をキャンセルするために使用されます。
  - `OPTIMIZE` を指定した場合、テーブル構造の最適化操作（バケット方式とバケット数の変更を含む）をキャンセルするために使用されます。
  - `ROLLUP` を指定した場合、Rollup インデックスの作成または削除操作をキャンセルするために使用されます。
- `db_name`：オプション。テーブルが存在するデータベース名。指定しない場合は、現在のデータベースがデフォルトになります。
- `table_name`：必須。テーブル名。

## 例

1. データベース `example_db` のテーブル `example_table` で行われている列の変更操作をキャンセルします。

   ```SQL
   CANCEL ALTER TABLE COLUMN FROM example_db.example_table;
   ```

2. データベース `example_db` のテーブル `example_table` で行われているテーブル構造の最適化操作をキャンセルします。

   ```SQL
   CANCEL ALTER TABLE OPTIMIZE FROM example_db.example_table;
   ```

3. 現在のデータベースで、`example_table` の Rollup インデックスの作成または削除操作をキャンセルします。

    ```SQL
    CANCEL ALTER TABLE ROLLUP FROM example_table;
    ```
