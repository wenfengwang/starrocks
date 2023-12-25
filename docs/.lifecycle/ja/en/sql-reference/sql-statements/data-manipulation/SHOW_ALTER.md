---
displayed_sidebar: English
---

# SHOW ALTER TABLE

## 説明

以下を含む進行中のALTER TABLE操作の実行を示します:

- 列の変更。
- テーブルスキーマの最適化 (v3.2から)、バケット化方法とバケット数の変更を含む。
- ロールアップインデックスの作成と削除。

## 構文

- 列の変更やテーブルスキーマの最適化の操作の実行を表示する。

    ```sql
    SHOW ALTER TABLE { COLUMN | OPTIMIZE } [FROM db_name] [WHERE TableName|CreateTime|FinishTime|State] [ORDER BY] [LIMIT]
    ```

- ロールアップインデックスの追加や削除の操作の実行を表示する。

    ```sql
    SHOW ALTER TABLE ROLLUP [FROM db_name]
    ```

## パラメータ

- `{COLUMN | OPTIMIZE | ROLLUP}`:

  - `COLUMN`が指定された場合、このステートメントは列の変更操作を表示します。
  - `OPTIMIZE`が指定された場合、このステートメントはテーブル構造の最適化操作を表示します。
  - `ROLLUP`が指定された場合、このステートメントはロールアップインデックスの追加や削除操作を表示します。

- `db_name`: 任意。`db_name`が指定されていない場合、デフォルトで現在のデータベースが使用されます。

## 例

1. 現在のデータベースでの列の変更、テーブルスキーマの最適化、ロールアップインデックスの作成や削除の操作の実行を表示します。

    ```sql
    SHOW ALTER TABLE COLUMN;
    SHOW ALTER TABLE OPTIMIZE;
    SHOW ALTER TABLE ROLLUP;
    ```

2. 指定されたデータベースでの列の変更、テーブルスキーマの最適化、ロールアップインデックスの作成や削除に関連する操作の実行を表示します。

    ```sql
    SHOW ALTER TABLE COLUMN FROM example_db;
    SHOW ALTER TABLE OPTIMIZE FROM example_db;
    SHOW ALTER TABLE ROLLUP FROM example_db;
    ```

3. 指定されたテーブルで最も最近の列の変更やテーブルスキーマの最適化の操作の実行を表示します。

    ```sql
    SHOW ALTER TABLE COLUMN WHERE TableName = "table1" ORDER BY CreateTime DESC LIMIT 1;
    SHOW ALTER TABLE OPTIMIZE WHERE TableName = "table1" ORDER BY CreateTime DESC LIMIT 1; 
    ```

## 参照

- [CREATE TABLE](../data-definition/CREATE_TABLE.md)
- [ALTER TABLE](../data-definition/ALTER_TABLE.md)
- [SHOW TABLES](../data-manipulation/SHOW_TABLES.md)
- [SHOW CREATE TABLE](../data-manipulation/SHOW_CREATE_TABLE.md)
