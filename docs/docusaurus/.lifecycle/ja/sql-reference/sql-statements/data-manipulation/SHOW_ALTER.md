---
displayed_sidebar: "Japanese"
---

# ALTER TABLEの表示

## 説明

実行中のALTER TABLEタスクの実行を表示します。

## 構文

```sql
SHOW ALTER TABLE {COLUMN | ROLLUP} [FROM <db_name>]
```

## パラメーター

- COLUMN | ROLLUP

  - COLUMNが指定された場合、このステートメントは列の修正タスクを表示します。WHERE句をネストする必要がある場合、サポートされている構文は`[WHERE TableName|CreateTime|FinishTime|State] [ORDER BY] [LIMIT]`です。

  - ROLLUPが指定された場合、このステートメントはROLLUPインデックスの作成または削除のタスクを示します。

- `db_name`：オプション。`db_name`が指定されていない場合、デフォルトで現在のデータベースが使用されます。

## 例

例1：現在のデータベースでの列の修正タスクを表示します。

```sql
SHOW ALTER TABLE COLUMN;
```

例2：テーブルの最新の列の修正タスクを表示します。

```sql
SHOW ALTER TABLE COLUMN WHERE TableName = "table1" ORDER BY CreateTime DESC LIMIT 1;
 ```

例3：指定されたデータベースでのROLLUPインデックスの作成または削除のタスクを表示します。

```sql
SHOW ALTER TABLE ROLLUP FROM example_db;
````

## リンク

- [CREATE TABLE](../data-definition/CREATE_TABLE.md)
- [ALTER TABLE](../data-definition/ALTER_TABLE.md)
- [SHOW TABLES](../data-manipulation/SHOW_TABLES.md)
- [SHOW CREATE TABLE](../data-manipulation/SHOW_CREATE_TABLE.md)