---
displayed_sidebar: English
---

# ALTER TABLE のキャンセル

## 説明

以下を含む進行中の ALTER TABLE 操作の実行をキャンセルします：

- 列の変更。
- テーブルスキーマの最適化 (v3.2 以降)、バケット化方法とバケット数の変更を含む。
- ロールアップインデックスの作成および削除。

> **注意**
>
> - このステートメントは同期操作です。
> - このステートメントを使用するには、テーブルに対する `ALTER_PRIV` 権限が必要です。
> - このステートメントは、前述のように ALTER TABLE を使用した非同期操作のキャンセルのみをサポートし、ALTER TABLE を使用した同期操作のキャンセル（rename など）はサポートしません。

## 構文

   ```SQL
   CANCEL ALTER TABLE { COLUMN | OPTIMIZE | ROLLUP } FROM [db_name.]table_name
   ```

## パラメータ

- `{COLUMN | OPTIMIZE | ROLLUP}`：

  - `COLUMN` が指定された場合、このステートメントは列の変更操作をキャンセルします。
  - `OPTIMIZE` が指定された場合、このステートメントはテーブルスキーマの最適化操作をキャンセルします。
  - `ROLLUP` が指定された場合、このステートメントはロールアップインデックスの追加または削除操作をキャンセルします。

- `db_name`：オプション。テーブルが属するデータベースの名前。このパラメータが指定されていない場合、デフォルトで現在のデータベースが使用されます。
- `table_name`：必須。テーブル名。

## 例

1. データベース `example_db` の `example_table` での列の変更操作をキャンセルします。

   ```SQL
   CANCEL ALTER TABLE COLUMN FROM example_db.example_table;
   ```

2. データベース `example_db` の `example_table` でのテーブルスキーマの最適化操作をキャンセルします。

   ```SQL
   CANCEL ALTER TABLE OPTIMIZE FROM example_db.example_table;
   ```

3. 現在のデータベースで `example_table` のロールアップインデックスの追加または削除操作をキャンセルします。

   ```SQL
   CANCEL ALTER TABLE ROLLUP FROM example_table;
   ```
