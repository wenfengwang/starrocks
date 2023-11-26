---
displayed_sidebar: "Japanese"
---

# SHOW ALTER TABLE

## 説明

進行中のALTER TABLEタスクの実行を表示します。

## 構文

```sql
SHOW ALTER TABLE {COLUMN | ROLLUP} [FROM <db_name>]
```

## パラメータ

- COLUMN | ROLLUP

  - COLUMNが指定された場合、このステートメントは列の変更タスクを表示します。WHERE句をネストする必要がある場合、サポートされる構文は `[WHERE TableName|CreateTime|FinishTime|State] [ORDER BY] [LIMIT]` です。

  - ROLLUPが指定された場合、このステートメントはROLLUPインデックスの作成または削除のタスクを表示します。

- `db_name`: オプションです。`db_name`が指定されていない場合、デフォルトで現在のデータベースが使用されます。

## 例

例1: 現在のデータベースの列変更タスクを表示します。

```sql
SHOW ALTER TABLE COLUMN;
```

例2: テーブルの最新の列変更タスクを表示します。

```sql
SHOW ALTER TABLE COLUMN WHERE TableName = "table1"
ORDER BY CreateTime DESC LIMIT 1;
 ```

例3: 指定されたデータベースのROLLUPインデックスの作成または削除のタスクを表示します。

```sql
SHOW ALTER TABLE ROLLUP FROM example_db;
````

## 参照

- [CREATE TABLE](../data-definition/CREATE_TABLE.md)
- [ALTER TABLE](../data-definition/ALTER_TABLE.md)
- [SHOW TABLES](../data-manipulation/SHOW_TABLES.md)
- [SHOW CREATE TABLE](../data-manipulation/SHOW_CREATE_TABLE.md)
