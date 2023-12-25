---
displayed_sidebar: Chinese
---

# ALTER TABLEの表示

## 機能

実行中のALTER TABLE操作の状況を確認することができます。内容には以下が含まれます：

- 列の変更。
- テーブル構造の最適化（バージョン3.2以降）、バケット方式とバケット数の変更を含む。
- Rollup Indexの作成と削除。

## 文法

- 列の変更またはテーブル構造の最適化操作の状況を確認する。

    ```sql
    SHOW ALTER TABLE { COLUMN | OPTIMIZE } [FROM <db_name>]
    [WHERE <where_condition> ] [ORDER BY <col_name> [ASC | DESC]] [LIMIT <num>]
    ```

- Rollup Indexの作成と削除操作の状況を確認する。

    ```sql
    SHOW ALTER TABLE ROLLUP [FROM <db_name>]
    ```

## パラメータ説明

- `{COLUMN | OPTIMIZE | ROLLUP}`：`COLUMN`、`OPTIMIZE`、`ROLLUP`の中から一つを選択する必要があります。
  - `COLUMN`を指定した場合、列の変更操作を確認するためにこの文を使用します。
  - `OPTIMIZE`を指定した場合、テーブル構造の最適化操作（バケット方式とバケット数の変更）を確認するためにこの文を使用します。
  - `ROLLUP`を指定した場合、Rollup Indexの作成または削除操作を確認するためにこの文を使用します。
- `COLUMN`または`OPTIMIZE`を指定して列の変更またはテーブル構造の最適化操作を確認する場合、以下の子句を使用できます：
  - `WHERE <where_condition>`：操作の`TableName`、`CreateTime`、`FinishTime`、`State`に基づいて条件を満たす操作をフィルタリングします。
  - `ORDER BY <col_name> [ASC | DESC]`：操作の`TableName`、`CreateTime`、`FinishTime`、`State`に基づいて結果セット内の操作を並べ替えます。
  - `LIMIT <num>`：指定された数の操作を返します。
- `db_name`：オプション。指定しない場合は、現在のデータベースがデフォルトで使用されます。

## 例

1. 現在のデータベースで実行中のすべての列の変更操作、テーブル構造の最適化操作、Rollup Indexの作成または削除操作の状況を確認する。

    ```sql
    SHOW ALTER TABLE COLUMN;
    SHOW ALTER TABLE OPTIMIZE;
    SHOW ALTER TABLE ROLLUP;
    ```

2. 指定されたデータベースで列の変更操作、テーブル構造の最適化操作、Rollup Indexの作成または削除操作の状況を確認する。

    ```sql
    SHOW ALTER TABLE COLUMN FROM example_db;
    SHOW ALTER TABLE OPTIMIZE FROM example_db;
    SHOW ALTER TABLE ROLLUP FROM example_db;
    ````

3. 特定のテーブルで最近行われた列の変更操作またはテーブル構造の最適化操作の状況を確認する。

    ```sql
    SHOW ALTER TABLE COLUMN WHERE TableName = "table1" ORDER BY CreateTime DESC LIMIT 1;
    SHOW ALTER TABLE OPTIMIZE WHERE TableName = "table1" ORDER BY CreateTime DESC LIMIT 1; 
    ```

## 関連参照

- [CREATE TABLE](../data-definition/CREATE_TABLE.md)
- [ALTER TABLE](../data-definition/ALTER_TABLE.md)
- [SHOW TABLES](../data-manipulation/SHOW_TABLES.md)
- [SHOW CREATE TABLE](../data-manipulation/SHOW_CREATE_TABLE.md)
